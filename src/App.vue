<script setup>
import { onMounted, ref, computed } from 'vue';

// --- 変数定義 ---
const content = ref('')
const author = ref('')
const translatedText = ref('')
const loading = ref(false)
const translating = ref(false)
const temp = ref(null)
const weatherCode = ref(null)
const copyLabel = ref('COPY TEXT')
const bgImage = ref('https://images.unsplash.com/photo-1464822759023-fed622ff2c3b?auto=format&fit=crop&w=1920&q=80')

// 天気を取得する関数（東京：北緯35.6895, 東経139.6917）
const fetchWeather = async () => {
  try {
    const res = await fetch('https://api.open-meteo.com/v1/forecast?latitude=35.6895&longitude=139.6917&current_weather=true')
    const data = await res.json()
    temp.value = data.current_weather.temperature
    weatherCode.value = data.current_weather.weathercode
  } catch (error) {
    console.error('天気の取得に失敗しました。')
  }
}

// APIからデータを取得する非同期関数
const fetchQuote = async () => {
  loading.value = true
  translatedText.value = '' // 新しい名言を取得するときは翻訳をリセット

  try {
    const res = await fetch('https://dummyjson.com/quotes/random')
    const data = await res.json()
    const keyword = data.quote.split(' ')[0]
    const newImageUrl = `https://loremflickr.com/1920/1080/${keyword}?lock=${Math.floor(Math.random() * 1000)}`

    // 画像の事前読み込み
    await new Promise((resolve) => {
      const img = new Image()
      img.src = newImageUrl
      img.onload = resolve // 画像のダウンロードが終わったら次に進む
      img.onerror = resolve // 失敗しても止まらないように進む
    })

    // 画像と文字を同時にセット
    content.value = data.quote // 名言
    author.value = data.author // 著書名
    bgImage.value = newImageUrl

  } catch (error) {
    content.value = '名言の取得に失敗しました。'
    console.error(error)
  } finally {
    loading.value = false
  }
}

// 翻訳を実行する
const translateQuote = async () => {
  if (!content.value) return
  translating.value = true
  try {
    const res = await fetch(`https://api.mymemory.translated.net/get?q=${encodeURIComponent(content.value)}&langpair=en|ja`)
    const data = await res.json()
    translatedText.value = data.responseData.translatedText
  } catch (error) {
    translatedText.value = `翻訳に失敗しました`
  } finally {
    translating.value = false
  }
}

const themeClass = computed(() => {
  if (temp.value === null) return 'theme-default';
  return temp.value >= 20 ? 'theme-warm' : 'theme-cool';
})

// 画面が表示された時に実行
onMounted(() => {
  fetchQuote()
  fetchWeather()
})

const copyToClipboard = async () => {
  const text =`"${content.value}" - ${author.value}`
  try {
    await navigator.clipboard.writeText(text)
    copyLabel.value = 'COPIED!'
    setTimeout(() => {
      copyLabel.value = 'COPY TEXT'
    }, 2000)
  } catch (error) {
    console.error('コピーに失敗しました。', error)
  }
}
</script>

<template>
  <main :class="['container', themeClass]" :style="{ backgroundImage: `linear-gradient(rgba(255,255,255,0.7), rgba(255,255,255,0.7)), url(${bgImage})` }">

    <div class="weather-badge" v-if="temp !== null">
      <span class="city">TOKYO</span>
      <span>{{ temp }}°C</span>
    </div>

    <div class="card">
      <h1 class="label">FETCH WISDOM</h1>
      <div class="quote-area">
        <p v-if="loading" class="loading">Fetching...</p>
        <div v-else class="quote-content">
          <p class="text">{{ content }}</p>
          <p v-if="translatedText" class="translated-text">{{ translatedText }}</p>
          <p class="author">— {{ author }}</p>
        </div>
      </div>
      <div class="button-group">
        <button @click="fetchQuote" :disabled="loading" class="btn">
          NEXT WISDOM
        </button>
        <button @click="translateQuote" :disabled="loading || translating" class="btn secondary">
          {{ translating ? 'TRANSLATING...' : 'TRANSLATE TO JP'  }}
        </button>
        <button @click="copyToClipboard" :disabled="loading || !content" class="btn secondary">
          {{ copyLabel }}
        </button>
      </div>
    </div>
  </main>
</template>

<style scoped>
.theme-default { background-color: #f8fafc; }
.theme-warm { background-color: #fff7ed; }
.theme-cool { background-color: #f0f9ff; }

.weather-badge {
  position: absolute;
  top: 2rem;
  right: 2rem;
  display: flex;
  flex-direction: column;
  align-items: flex-end;
  font-size: 0.7rem;
  letter-spacing: 0.1em;
  color: #94a3b8;
}

.city {
  font-weight: bold;
  border-bottom: 1px solid #e2e8f0;
  margin-bottom: 2px;
}

.container {
  min-height: 100vh;
  display: flex;
  align-items: center;
  justify-content: center;
  font-family: ui-monospace, monospace;
  transition: background-color 0.5s ease;
  background-size: cover;
  background-position: center;
  transition: background-color 0.8s ease, background-image 1.0s ease;
}

.card {
  text-align: center;
  max-width: 600px;
  padding: 2rem;
}

.label {
  font-size: 0.75rem;
  letter-spacing: 0.3em;
  color: #94a3b8;
  margin-bottom: 2rem;
}

.quote-area {
  min-height: 200px;
  display: flex;
  align-items: center;
  justify-content: center;
}

.text {
  font-family: serif;
  font-style: italic;
  font-size: 1.6rem;
  color: #1e293b;
  line-height: 1.4;
  margin-bottom: 1rem;
}

.translated-text {
  font-family: sans-serif;
  font-size: 1rem;
  color: #475569;
  margin-bottom: 1.5rem;
  line-height: 1.6;
  background-color: #f1f5f9;
  padding: 1rem;
  border-radius: 8px;
}

.author {
  font-size: 0.85rem;
  color: #64748b;
  letter-spacing: 0.1em;
  text-transform: uppercase;
}

/* ボタンエリア */
.button-group {
  display: flex;
  gap: 1rem;
  justify-content: center;
  margin-top: 3rem;
}

.btn {
  padding: 0.5rem 1.5rem;
  border: 1px solid #0f172a;
  background: none;
  cursor: pointer;
  font-size: 0.75rem;
  letter-spacing: 0.1em;
  transition: all 0.3s;
}

.btn.secondary {
  border-color: #94a3b8;
  color: #64748b;
}

.btn:hover:not(:disabled) {
  background-color: #0f172a;
  color: white;
}

.btn.secondary:hover:not(:disabled) {
  background-color: #f1f5f9;
  color: #1e293b;
  border-color: #1e293b;
}

.btn:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

/* アニメーション */
@keyframes pulse {
  50% { opacity: .5; }
}

.loading {
  animation: pulse 2s infinite;
  color: #94a3b8;
}
</style>