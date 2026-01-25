<script setup>
import { onMounted, ref } from 'vue';

// 状態（リアクティブな変数）を定義
const quote = ref('')
const loading = ref(true)

// APIからデータを取得する非同期関数
const fetchQuote = async () => {
  loading.value = true
  try {
    const res = await fetch('https://api.adviceslip.com/advice')
    const data = await res.json()
    quote.value = data.slip.advice
  } catch (error) {
    quote.value = '知恵の取得に失敗しました'
  } finally {
    loading.value = false
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
        <p v-if="loading" class="loading">Fetching</p>
        <p v-else class="quote-text">"{{ quote }}"</p>
      </div>
      <button @click="fetchQuote" :disabled="loading" class="btn">
        NEXT WISDOM
      </button>
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
  max-width: 500px;
  padding: 2rem;
}

.label {
  font-size: 0.75rem;
  letter-spacing: 0.3em;
  color: #94a3b8;
  margin-bottom: 2rem;
}

.quote-area {
  min-height: 120px;
  display: flex;
  align-items: center;
  justify-content: center;
}

.quote-text {
  font-family: serif;
  font-style: italic;
  font-size: 1.5rem;
  color: #1e293b;
  line-height: 1.6;
}

.btn {
  margin-top: 2rem;
  padding: 0.5rem 1.5rem;
  border: 1px solid #0f172a;
  background: none;
  cursor: pointer;
  font-size: 0.75rem;
  letter-spacing: 0.1em;
  transition: all 0.3s;
}

.btn:hover:not(:disabled) {
  background-color: #0f172a;
  color: white;
}

.btn:disabled {
  opacity: 0.5;
}

@keyframes pulse {
  50% { opacity: .5; }
}
.loading {
  animation: pulse 2s infinite;
}
</style>