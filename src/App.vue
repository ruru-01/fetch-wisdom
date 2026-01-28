<script setup>
import { onMounted, ref } from 'vue';

// 状態（リアクティブな変数）を定義
const content = ref('')
const author = ref('')
const translatedText = ref('')
const loading = ref(false)
const translating = ref(false)

// APIからデータを取得する非同期関数
const fetchQuote = async () => {
  loading.value = true
  translatedText.value = '' // 新しい名言を取得するときは翻訳をリセット
  try {
    const res = await fetch('https://dummyjson.com/quotes/random')
    const data = await res.json()
    content.value = data.quote // 名言
    author.value = data.author // 著書名
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

// 画面が表示された時に実行
onMounted(fetchQuote)
</script>

<template>
  <main class="container">
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
      </div>
    </div>
  </main>
</template>

<style scoped>
.container {
  min-height: 100vh;
  display: flex;
  align-items: center;
  justify-content: center;
  background-color: #F9F8F6;
  font-family: ui-monospace, monospace;
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