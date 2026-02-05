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
const copyLabel = ref('こっそり持ち帰る')
const bgImage = ref('https://images.unsplash.com/photo-1464822759023-fed622ff2c3b?auto=format&fit=crop&w=1920&q=80')
const favorites = ref(JSON.parse(localStorage.getItem('myFavoriteQuotes') || '[]'))

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
    copyLabel.value = '持ち帰りました!'
    setTimeout(() => {
      copyLabel.value = 'こっそり持ち帰る'
    }, 2000)
  } catch (error) {
    console.error('失敗しちゃいました。', error)
  }
}

// お気に入りに追加する関数
const addToFavorites = () => {
  const newFav = {
    content: content.value,
    author: author.value,
    translated: translatedText.value
  }

  // 重複チェック（同じ名言がなければ追加）
  if(!favorites.value.some(f => f.content === newFav.content)) {
    favorites.value.push(newFav)
    localStorage.setItem('myFavoriteQuotes', JSON.stringify(favorites.value))
  }
}

// お気に入りから削除する関数
const removeFavorite = (index) => {
  favorites.value.splice(index, 1)
  localStorage.setItem('myFavoriteQuotes', JSON.stringify(favorites.value))
}

</script>

<template>
  <main :class="['container', themeClass]" :style="{ backgroundImage: `linear-gradient(rgba(255,255,255,0.7), rgba(255,255,255,0.7)), url(${bgImage})` }">
    <div class="weather-badge" v-if="temp !== null">
      <span class="city">本日の東京のご機嫌</span>
      <span>{{ temp }}°C</span>
    </div>

    <div class="card-wrapper">
      <div class="card">
        <h1 class="label">今日の魂に響く名言</h1>

        <div class="quote-area">
          <p v-if="loading" class="loading">ちょっと待って、今いいこと言うから...</p>
          <div v-else class="quote-content">
            <p class="text">{{ content }}</p>
            <p v-if="translatedText" class="translated-text">{{ translatedText }}</p>
            <p class="author">— {{ author }}</p>
          </div>
        </div>

        <div class="button-group">
          <button @click="fetchQuote" :disabled="loading" class="btn">
            次のお告げを聞く
          </button>
          <button @click="translateQuote" :disabled="loading || translating" class="btn secondary">
            {{ translating ? '魔法をかけてます...' : '魔法の翻訳機を起動！'  }}
          </button>
          <button @click="copyToClipboard" :disabled="loading || !content" class="btn secondary">
            {{ copyLabel }}
          </button>
          <button @click="addToFavorites" :disabled="loading || !content" class="btn secondary">
            一生忘れない
          </button>
        </div>
      </div>

      <div v-if="favorites.length > 0" class="favorites-section">
        <h3>宝箱にしまった言葉たち</h3>
        <ul class="fav-list">
          <li v-for="(fav, index) in favorites" :key="index" class="fav-item">
            <p>"{{ fav.content }}"</p>
            <button @click="removeFavorite(index)" class="delete-btn">捨てちゃう</button>
          </li>
        </ul>
      </div>
    </div>



  </main>
</template>

<style scoped>
.theme-default { background-color: #f8fafc; }
.theme-warm { background-color: #fff7ed; }
.theme-cool { background-color: #f0f9ff; }

/** --- 天気バッジ --- */
.weather-badge {
  position: fixed;
  top: 1.5rem;
  right: 1.5rem;
  padding: 0.8rem 1.2rem;
  background: rgba(255, 255, 255, 0.2);
  backdrop-filter: blur(10px);
  border-radius: 50px;
  border: 1px solid rgba(255, 255, 255, 0.3);
  font-size: 0.8rem;
  color: #1e293b;
  display: flex;
  gap: 8px;
  box-shadow: 0 4px 12px rgba(0,0,0,0.05);
  z-index: 100;
}

.city {
  font-weight: bold;
  border-bottom: 1px solid #e2e8f0;
  margin-bottom: 2px;
}

/** --- メインカード --- */
.card-wrapper {
  display: flex;
  flex-direction: column;
  align-items: center;
  width: 90%;
  max-width: 1000px;
}

.fav-item:hover {
  transform: scale(1.02);
}

.delete-btn {
  background: none;
  border: none;
  color: #ef4444;
  cursor: pointer;
  font-size: 1.2rem;
}

/** --- レイアウト基盤 --- */
.container {
  min-height: 100vh;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: flex-start; /* 上から順に並べる */
  padding-top: 5vh;
  font-family: 'Helvetica Neue', Arial, 'Hiragino Kaku Gothic ProN', 'Hiragino Sans', Meiryo, sans-serif;
  background-size: cover;
  background-position: center;
  background-attachment: fixed; /* 背景を固定して高級感を出す */
  transition: background-image 1.2s ease-in-out;
}

.card {
  text-align: center;
  width: 100%;
  max-width: 650px;
  padding: 4rem 2rem;
  background: rgba(255, 255, 255, 0.1);
  backdrop-filter: blur(20px);
  -webkit-backdrop-filter: blur(20px);
  border: 1px solid rgba(255, 255, 255, 0.4);
  border-radius: 32px;
  box-shadow: 0 20px 50px rgba(0, 0, 0, 0.1);

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

/** --- 名言エリア --- */
.text {
  font-family: "Times New Roman", serif;
  font-style: italic;
  font-size: 2rem;
  line-height: 1.6;
  color: #0f172a;
  margin-bottom: 2rem;
  text-shadow: 0 2px 10px rgba(255, 255, 255, 0.8);
}

.translated-text {
  font-size: 1.1rem;
  color: #334155;
  background: rgba(255, 255, 255, 0.5);
  padding: 1.5rem;
  border-radius: 20px;
  margin: 1.5rem 0;
  border: 1px solid rgba(255, 255, 255, 0.5);
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
  flex-wrap: wrap;
  gap: 12px;
  justify-content: center;
  margin-top: 3rem;
}

.btn {
  padding: 0.8rem 1.8rem;
  border: none;
  border-radius: 16px;
  background: #0f172a;
  color: white;
  font-weight: bold;
  cursor: pointer;
  transition: all 0.4s cubic-bezier(0.175, 0.885, 0.32, 1.275);
  box-shadow: 0 4px 15px rgba(0,0,0,0.2);
}

.btn:hover:not(:disabled) {
  transform: scale(1.08) translateY(-5px);
  box-shadow: 0 12px 25px rgba(0,0,0,0.3);
}

.btn.secondary {
  background: rgba(255, 255, 255, 0.8);
  color: #0f172a;
  border: 1px solid rgba(15, 23, 42, 0.1);
}

.btn.secondary:hover:not(:disabled) {
  background: #f8fafc;
  border-color: #0f172a;
}

.btn:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

/** --- お気に入りセクション --- */
.favorites-section {
  margin-top: 5rem;
  width: 100%;
}

.favorites-section h3 {
  text-align: center;
  color: #1e293b;
  letter-spacing: 0.2em;
  margin-bottom: 2rem;
  font-size: 1.2rem;
}

.fav-list {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(280px, 1fr)); /* ここでタイル状に */
  gap: 20px;
  padding: 0;
  list-style: none;
}

.fav-item {
  background: rgba(255, 255, 255, 0.3);
  backdrop-filter: blur(10px);
  padding: 1.5rem;
  border-radius: 24px;
  border: 1px solid rgba(255, 255, 255, 0.4);
  display: flex;
  flex-direction: column; /* 縦並びに */
  justify-content: space-between;
  transition: all 0.3s ease;
  min-height: 150px;
}

.fav-item:hover {
  transform: translateY(-10px) rotate(1deg); /* 少し傾ける遊び心 */
  background: rgba(255, 255, 255, 0.5);
}

.fav-item p {
  font-size: 0.9rem;
  line-height: 1.5;
  color: #1e293b;
  margin: 0 0 1rem 0;
}

.delete-btn {
  align-self: flex-end;
  background: #fee2e2;
  color: #ef4444;
  border: none;
  padding: 4px 12px;
  border-radius: 10px;
  font-size: 0.75rem;
  cursor: pointer;
  transition: 0.2s;
}

.delete-btn:hover {
  background: #ef4444;
  color: white;
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