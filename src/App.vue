<template>
  <div class="container">
    <header>
      <h1>🧠 Vue Matching Cards</h1>
      <div class="controls">
        <select id="categories" v-model="query" @change="resetGame">
          <option id="nature" value="nature">Nature</option>
          <option id="animals" value="animals">Animals</option>
          <option id="travel" value="travel">Travel</option>
          <option id="technology" value="technology">Technology</option>
          <option id="food" value="food">Food</option>
        </select>
        <select id="size" v-model.number="pairs" @change="resetGame">
          <option id="v6" :value="6">6 pairs (12 cards)</option>
          <option id="v8" :value="8">8 pairs (16 cards)</option>
          <option id="v10" :value="10">10 pairs (20 cards)</option>
        </select>
        <button @click="resetGame">🔄 Reset</button>
      </div>
    </header>

    <div class="stats">
      <div>Moves: <span class="badge">{{ moves }}</span></div>
      <div>Matches: <span class="badge">{{ matches }}/{{ pairs }}</span></div>
      <div v-if="timerRunning">Time: <span class="badge">{{ elapsedDisplay }}</span></div>
    </div>

    <div class="grid">
      <div
        class="card"
        v-for="(card, idx) in cards"
        :key="card.uid"
        :class="{ flipped: card.flipped || card.matched }"
        @click="onFlip(idx)"
      >
        <div class="inner">
          <div class="face">
            <img :src="card.url" :alt="'card-' + idx" referrerpolicy="no-referrer" />
          </div>
          <div class="back">MATCH</div>
        </div>
      </div>
    </div>

    <footer>
      Images provided by Unsplash. Categories: {{ query }}.
    </footer>
  </div>
</template>

<script setup>
import { ref, computed, onMounted, onBeforeUnmount } from 'vue'

const accessKey = import.meta.env.VITE_UNSPLASH_ACCESS_KEY
const query = ref('nature')
const pairs = ref(8)

const cards = ref([])
const moves = ref(0)
const matches = ref(0)
const flipping = ref(false)
const flippedIndices = ref([])

// timer
const timer = ref(0)
const timerRunning = ref(false)
let intervalId = null
const elapsedDisplay = computed(() => {
  const s = Math.floor(timer.value / 1000)
  const mm = String(Math.floor(s / 60)).padStart(2, '0')
  const ss = String(s % 60).padStart(2, '0')
  return mm + ':' + ss
})

function startTimer() {
  if (timerRunning.value) return
  timerRunning.value = true
  const start = Date.now() - timer.value
  intervalId = setInterval(() => {
    timer.value = Date.now() - start
  }, 250)
}

function stopTimer() {
  timerRunning.value = false
  if (intervalId) clearInterval(intervalId)
}

function shuffle(array) {
  for (let i = array.length - 1; i > 0; i--) {
    const j = Math.floor(Math.random() * (i + 1))
    ;[array[i], array[j]] = [array[j], array[i]]
  }
  return array
}

async function fetchUnsplashImages(topic, count) {
  if (!accessKey) {
    console.warn('Missing VITE_UNSPLASH_ACCESS_KEY. Using placeholder images.')
    // Placeholder via picsum (no key) as a fallback
    const urls = Array.from({ length: count }, (_, i) => `https://picsum.photos/seed/${topic}-${i}/300/300`)
    return urls
  }

  const perPage = Math.max(8, count) // fetch at least count images
  const url = new URL('https://api.unsplash.com/search/photos')
  url.searchParams.set('query', topic)
  url.searchParams.set('per_page', perPage.toString())
  url.searchParams.set('orientation', 'squarish')
  url.searchParams.set('content_filter', 'high')

console.log(`Fetching ${count} images for topic "${topic}" from Unsplash...`)
  console.log(`Using access key: ${accessKey}`)
  console.log(`Request URL: ${url}`)

  const res = await fetch(url, {
    headers: {
      Authorization: `Client-ID ${accessKey}`
    }
  })
  if (!res.ok) {
    console.warn('Unsplash fetch failed, falling back to placeholders.')
    const urls = Array.from({ length: count }, (_, i) => `https://picsum.photos/seed/${topic}-${i}/300/300`)
    console.log(`Using placeholder images: ${urls}`)
    return urls
  }
  const data = await res.json()
  console.log(`Fetched ${data.results.length} images from Unsplash.`)
  const urls = (data.results || []).slice(0, count).map(r => r.urls && (r.urls.small_s3 || r.urls.small || r.urls.thumb)).filter(Boolean)

  console.log(`Using ${urls} images for the game.`)
  // If not enough images, fill with placeholders
  while (urls.length < count) {
    urls.push(`https://picsum.photos/seed/${topic}-${urls.length}/300/300`)
  }
  console.log(`Fetched ${urls} images for topic "${topic}".`)
  return urls
}

async function setupGame() {
  moves.value = 0
  matches.value = 0
  flippedIndices.value = []
  flipping.value = false
  stopTimer()
  timer.value = 0

  const urls = await fetchUnsplashImages(query.value, pairs.value)
  const deck = urls.flatMap((u, i) => [
    { uid: `a-${i}`, url: u, flipped: false, matched: false, pair: i },
    { uid: `b-${i}`, url: u, flipped: false, matched: false, pair: i }
  ])

  console.log(JSON.stringify(deck, null, 2) + `\nDeck size: ${deck.length} cards.`)

  cards.value = shuffle(deck)
}

function onFlip(idx) {
  if (flipping.value) return
  const card = cards.value[idx]
  if (card.matched || card.flipped) return

  startTimer()
  card.flipped = true
  flippedIndices.value.push(idx)

  if (flippedIndices.value.length === 2) {
    moves.value++
    const [i1, i2] = flippedIndices.value
    const c1 = cards.value[i1]
    const c2 = cards.value[i2]

    if (c1.pair === c2.pair) {
      c1.matched = true
      c2.matched = true
      matches.value++
      flippedIndices.value = []

      if (matches.value === pairs.value) {
        stopTimer()
      }
    } else {
      flipping.value = true
      setTimeout(() => {
        c1.flipped = false
        c2.flipped = false
        flippedIndices.value = []
        flipping.value = false
      }, 900)
    }
  }
}

function resetGame() {
  setupGame()
}

onMounted(() => {
  setupGame()
})
onBeforeUnmount(() => stopTimer())
</script>
