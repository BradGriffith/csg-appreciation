<template>
  <div class="pictogram-container">
    <div class="max-w-4xl mx-auto p-8">
      <h1 class="title">
        Faculty & Staff Appreciation
      </h1>
      <p class="tagline">She will know her power.</p>

      <!-- Statistics Display -->
      <div class="stats-card">
        <div class="text-center">
          <div v-if="isLoading" class="stat-loading">
            <div class="loading-spinner"></div>
            <div class="loading-text-large">Loading...</div>
          </div>
          <div
            v-else
            class="stat-number"
            :title="`${vouchersPurchased} of ${goalVouchers} vouchers purchased`"
          >
            {{ supportPercentage }}%
          </div>
          <div class="stat-label">
            of Faculty & Staff supported
          </div>
        </div>
      </div>

      <!-- Pictogram Grid -->
      <div class="pictogram-card">
        <div class="icon-grid">
          <TeacherIcon
            v-for="(item, index) in iconData"
            :key="index"
            :fill-percentage="item.fillPercentage"
            :size="iconSize"
            fill-color="#c8102e"
            stroke-color="#c8102e"
            class="icon-item"
          />
        </div>
      </div>

      <!-- Testing Control -->
      <div v-if="isTestMode" class="control-card">
        <label class="control-label">
          TEST: Vouchers Purchased
        </label>
        <div class="counter-control">
          <button @click="decrementVouchers" class="counter-btn">−</button>
          <input
            v-model.number="vouchersPurchased"
            type="number"
            :min="0"
            :max="goalVouchers"
            class="counter-input"
          />
          <button @click="incrementVouchers" class="counter-btn">+</button>
        </div>
        <div class="slider-labels">
          <span>0</span>
          <span>{{ goalVouchers }}</span>
        </div>

        <!-- Quick Select Buttons -->
        <div class="button-group">
          <button @click="vouchersPurchased = 0" class="btn btn-outline">
            0
          </button>
          <button @click="vouchersPurchased = Math.floor(goalVouchers / 4)" class="btn btn-yellow">
            {{ Math.floor(goalVouchers / 4) }}
          </button>
          <button @click="vouchersPurchased = Math.floor(goalVouchers / 2)" class="btn btn-teal">
            {{ Math.floor(goalVouchers / 2) }}
          </button>
          <button @click="vouchersPurchased = Math.floor(goalVouchers * 0.75)" class="btn btn-green">
            {{ Math.floor(goalVouchers * 0.75) }}
          </button>
          <button @click="vouchersPurchased = goalVouchers" class="btn btn-red">
            {{ goalVouchers }}
          </button>
        </div>
      </div>

      <!-- Footer with Refresh Info -->
      <div class="footer-card">
        <div v-if="errorMessage" class="error-message">
          {{ errorMessage }}
        </div>
        <div class="footer-content">
          <div class="last-updated">
            <span v-if="isLoading" class="loading-text">Loading...</span>
            <span v-else-if="lastRefreshed" class="update-time">
              Last updated: {{ formatDate(lastRefreshed) }}
            </span>
          </div>
          <button
            @click="handleRefresh"
            :disabled="isLoading"
            class="refresh-btn"
            :class="{ 'refreshing': isLoading }"
          >
            <svg
              xmlns="http://www.w3.org/2000/svg"
              width="16"
              height="16"
              viewBox="0 0 24 24"
              fill="none"
              stroke="currentColor"
              stroke-width="2"
              stroke-linecap="round"
              stroke-linejoin="round"
              class="refresh-icon"
            >
              <path d="M21.5 2v6h-6M2.5 22v-6h6M2 11.5a10 10 0 0 1 18.8-4.3M22 12.5a10 10 0 0 1-18.8 4.2"/>
            </svg>
            {{ isLoading ? 'Refreshing...' : 'Refresh Data' }}
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { computed, ref, onMounted } from 'vue'
import TeacherIcon from './TeacherIcon.vue'

const totalIcons = 100
const goalVouchers = 220
const vouchersPurchased = ref(0)
const iconSize = 40
const isLoading = ref(true)
const lastRefreshed = ref(null)
const errorMessage = ref(null)

// Check if test mode is enabled via query parameter
const isTestMode = ref(false)
if (typeof window !== 'undefined') {
  const urlParams = new URLSearchParams(window.location.search)
  isTestMode.value = urlParams.get('test') === '1'
}

const API_URL = import.meta.env.VITE_API_URL
const CACHE_KEY = 'teacher_appreciation_data'
const CACHE_DURATION = 60 * 60 * 1000 // 1 hour in milliseconds

// Format date for display
const formatDate = (date) => {
  return new Date(date).toLocaleString('en-US', {
    month: 'short',
    day: 'numeric',
    year: 'numeric',
    hour: 'numeric',
    minute: '2-digit',
    hour12: true
  })
}

// Check if cached data is still valid
const getCachedData = () => {
  try {
    const cached = localStorage.getItem(CACHE_KEY)
    if (!cached) return null

    const { data, timestamp } = JSON.parse(cached)
    const now = Date.now()

    // Check if cache is less than 1 hour old
    if (now - timestamp < CACHE_DURATION) {
      return { data, timestamp }
    }

    // Cache expired
    return null
  } catch (error) {
    console.error('Error reading cache:', error)
    return null
  }
}

// Save data to cache
const setCachedData = (data) => {
  try {
    const timestamp = Date.now()
    localStorage.setItem(CACHE_KEY, JSON.stringify({ data, timestamp }))
    return timestamp
  } catch (error) {
    console.error('Error saving to cache:', error)
    return Date.now()
  }
}

// Fetch voucher data from API
const fetchVoucherData = async (forceRefresh = false) => {
  // Check cache first unless forcing refresh
  if (!forceRefresh) {
    const cached = getCachedData()
    if (cached) {
      vouchersPurchased.value = cached.data.item_count
      lastRefreshed.value = cached.timestamp
      isLoading.value = false
      errorMessage.value = null
      return
    }
  }

  isLoading.value = true
  errorMessage.value = null

  try {
    const response = await fetch(API_URL)

    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`)
    }

    const data = await response.json()

    // Extract item_count from the first product in by_product array
    const itemCount = data.summary?.by_product?.[0]?.item_count || 0

    vouchersPurchased.value = itemCount
    const timestamp = setCachedData({ item_count: itemCount })
    lastRefreshed.value = timestamp
  } catch (error) {
    console.error('Error fetching voucher data:', error)
    errorMessage.value = 'Unable to load voucher data. Using cached data if available.'

    // Try to use cached data even if expired
    const cached = localStorage.getItem(CACHE_KEY)
    if (cached) {
      const { data, timestamp } = JSON.parse(cached)
      vouchersPurchased.value = data.item_count
      lastRefreshed.value = timestamp
    }
  } finally {
    isLoading.value = false
  }
}

// Handle manual refresh
const handleRefresh = async () => {
  await fetchVoucherData(true)
}

// Fetch data on component mount
onMounted(() => {
  fetchVoucherData()
})

// Calculate percentage of goal reached
const supportPercentage = computed(() => {
  return Math.round((vouchersPurchased.value / goalVouchers) * 100)
})

// Calculate how many vouchers each icon represents
const vouchersPerIcon = goalVouchers / totalIcons // 2.2 vouchers per icon

// Generate icon data with partial fill from bottom to top
const iconData = computed(() => {
  const icons = []
  const vouchersFilled = vouchersPurchased.value

  for (let i = 0; i < totalIcons; i++) {
    // Calculate from bottom up (reverse order)
    const iconIndex = totalIcons - 1 - i
    const vouchersAtThisIcon = iconIndex * vouchersPerIcon
    const vouchersRemaining = vouchersFilled - vouchersAtThisIcon

    let fillPercentage = 0
    if (vouchersRemaining >= vouchersPerIcon) {
      fillPercentage = 100 // Fully filled
    } else if (vouchersRemaining > 0) {
      fillPercentage = (vouchersRemaining / vouchersPerIcon) * 100 // Partially filled
    }

    icons.push({
      fillPercentage: Math.max(0, Math.min(100, fillPercentage))
    })
  }

  return icons
})

const incrementVouchers = () => {
  if (vouchersPurchased.value < goalVouchers) {
    vouchersPurchased.value++
  }
}

const decrementVouchers = () => {
  if (vouchersPurchased.value > 0) {
    vouchersPurchased.value--
  }
}
</script>

<style scoped>
@import url('https://fonts.googleapis.com/css2?family=Lora:wght@400;700&family=Nunito+Sans:wght@300;400;600;700;800;900&display=swap');

.pictogram-container {
  min-height: 100vh;
  background: linear-gradient(135deg, #c8102e 0%, #9d7ad2 100%);
  padding: 1rem 0;
  font-family: 'Nunito Sans', sans-serif;
}

.title {
  color: #ffffff;
  font-family: 'Lora', serif;
  font-size: 56px;
  font-weight: 400;
  line-height: 120%;
  text-align: center;
  margin: 0 0 10px;
}

.tagline {
  color: #ffc600;
  font-family: 'Nunito Sans', sans-serif;
  font-size: 28px;
  font-weight: 300;
  text-align: center;
  margin: 0 0 20px;
}

.stats-card,
.pictogram-card,
.control-card {
  background: white;
  border-radius: 8px;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
  margin-bottom: 1rem;
}

.stats-card {
  padding: 1rem;
}

.stat-number {
  font-size: 3rem;
  font-weight: 700;
  color: #c8102e;
  margin-bottom: 0.25rem;
  font-family: 'Nunito Sans', sans-serif;
  cursor: help;
}

.stat-loading {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 1rem;
  padding: 1rem 0;
}

.loading-spinner {
  width: 48px;
  height: 48px;
  border: 4px solid #f3f3f3;
  border-top: 4px solid #c8102e;
  border-radius: 50%;
  animation: spin 1s linear infinite;
}

.loading-text-large {
  font-size: 1.5rem;
  font-weight: 600;
  color: #c8102e;
  font-family: 'Nunito Sans', sans-serif;
}

.stat-label {
  font-size: 1rem;
  color: #242424;
  font-weight: 600;
}

.stat-sublabel {
  font-size: 0.875rem;
  color: #636363;
  margin-top: 0.25rem;
}

.pictogram-card {
  padding: 1rem;
}

.icon-grid {
  display: flex;
  flex-wrap: wrap;
  gap: 0.25rem;
  max-width: 100%;
  justify-content: center;
}

.icon-item {
  /* Icons size themselves naturally */
}

.control-card {
  padding: 1.5rem;
}

.control-label {
  display: block;
  font-size: 1.125rem;
  font-weight: 700;
  color: #242424;
  margin-bottom: 1rem;
  font-family: 'Nunito Sans', sans-serif;
}

.counter-control {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 1rem;
  margin-bottom: 1rem;
}

.counter-btn {
  width: 48px;
  height: 48px;
  border: 2px solid #c8102e;
  border-radius: 8px;
  background: white;
  color: #c8102e;
  font-size: 24px;
  font-weight: 700;
  cursor: pointer;
  transition: all 0.2s ease;
  display: flex;
  align-items: center;
  justify-content: center;
}

.counter-btn:hover {
  background: #c8102e;
  color: white;
}

.counter-btn:active {
  transform: scale(0.95);
}

.counter-input {
  width: 120px;
  height: 48px;
  border: 2px solid #e5e5e5;
  border-radius: 8px;
  text-align: center;
  font-size: 1.5rem;
  font-weight: 700;
  color: #242424;
  font-family: 'Nunito Sans', sans-serif;
}

.counter-input:focus {
  outline: none;
  border-color: #c8102e;
}

.slider {
  width: 100%;
  height: 12px;
  background: #e5e5e5;
  border-radius: 0.5rem;
  appearance: none;
  cursor: pointer;
}

.slider::-webkit-slider-thumb {
  appearance: none;
  width: 24px;
  height: 24px;
  border-radius: 50%;
  background: #c8102e;
  cursor: pointer;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.2);
  transition: all 0.2s ease;
}

.slider::-webkit-slider-thumb:hover {
  background: #a00d25;
  transform: scale(1.1);
}

.slider::-moz-range-thumb {
  width: 24px;
  height: 24px;
  border-radius: 50%;
  background: #c8102e;
  cursor: pointer;
  border: none;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.2);
  transition: all 0.2s ease;
}

.slider::-moz-range-thumb:hover {
  background: #a00d25;
  transform: scale(1.1);
}

.slider-labels {
  display: flex;
  justify-content: space-between;
  font-size: 0.875rem;
  color: #636363;
  margin-top: 0.5rem;
}

.button-group {
  display: flex;
  gap: 0.5rem;
  margin-top: 1rem;
  flex-wrap: wrap;
  justify-content: center;
}

.btn {
  display: inline-block;
  border: 1px solid;
  padding: 15px 30px;
  font-family: 'Nunito Sans', sans-serif;
  font-size: 14px;
  font-weight: 700;
  line-height: 19px;
  text-align: center;
  text-decoration: none;
  cursor: pointer;
  transition: all 0.3s ease;
  border-radius: 4px;
}

.btn-outline {
  border-color: #e5e5e5;
  background: #f8f8f8;
  color: #242424;
}

.btn-outline:hover {
  background: white;
  color: #c8102e;
}

.btn-yellow {
  border-color: #ffc600;
  background: #ffc600;
  color: #242424;
}

.btn-yellow:hover {
  border-color: #242424;
  background: #242424;
  color: #ffc600;
}

.btn-teal {
  border-color: #40c1bb;
  background: #40c1bb;
  color: #242424;
}

.btn-teal:hover {
  border-color: #242424;
  background: #242424;
  color: #40c1bb;
}

.btn-green {
  border-color: #73c92d;
  background: #73c92d;
  color: #242424;
}

.btn-green:hover {
  border-color: #242424;
  background: #242424;
  color: #73c92d;
}

.btn-red {
  border-color: #c8102e;
  background: #c8102e;
  color: white;
}

.btn-red:hover {
  border-color: #c8102e;
  background: white;
  color: #c8102e;
}

.footer-card {
  background: white;
  border-radius: 8px;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
  padding: 1rem;
  margin-top: 1rem;
}

.error-message {
  background: #fef2f2;
  border: 1px solid #fecaca;
  color: #991b1b;
  padding: 0.75rem;
  border-radius: 6px;
  margin-bottom: 1rem;
  font-size: 0.875rem;
  font-weight: 600;
}

.footer-content {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 1rem;
  flex-wrap: wrap;
}

.last-updated {
  color: #636363;
  font-size: 0.875rem;
}

.loading-text {
  color: #c8102e;
  font-weight: 600;
}

.update-time {
  font-family: 'Nunito Sans', sans-serif;
}

.refresh-btn {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  padding: 10px 20px;
  background: #c8102e;
  color: white;
  border: 2px solid #c8102e;
  border-radius: 6px;
  font-family: 'Nunito Sans', sans-serif;
  font-size: 14px;
  font-weight: 700;
  cursor: pointer;
  transition: all 0.3s ease;
}

.refresh-btn:hover:not(:disabled) {
  background: white;
  color: #c8102e;
}

.refresh-btn:disabled {
  opacity: 0.6;
  cursor: not-allowed;
}

.refresh-btn.refreshing .refresh-icon {
  animation: spin 1s linear infinite;
}

.refresh-icon {
  flex-shrink: 0;
}

@keyframes spin {
  from {
    transform: rotate(0deg);
  }
  to {
    transform: rotate(360deg);
  }
}

@media (max-width: 768px) {
  .pictogram-container {
    padding: 0.25rem 0;
  }

  .max-w-4xl {
    padding: 0.75rem;
  }

  .title {
    font-size: 22px;
    margin: 0 0 4px;
    line-height: 100%;
  }

  .tagline {
    font-size: 14px;
    margin: 0 0 10px;
  }

  .stats-card {
    padding: 0.75rem;
    margin-bottom: 0.5rem;
  }

  .stat-number {
    font-size: 2rem;
    margin-bottom: 0.1rem;
  }

  .loading-spinner {
    width: 36px;
    height: 36px;
    border-width: 3px;
  }

  .loading-text-large {
    font-size: 1.125rem;
  }

  .stat-label {
    font-size: 0.75rem;
  }

  .stat-sublabel {
    font-size: 0.65rem;
    margin-top: 0.1rem;
  }

  .pictogram-card {
    padding: 0.5rem;
    margin-bottom: 0.5rem;
  }

  .icon-grid {
    gap: 0.15rem;
  }

  .person-icon {
    width: 35px;
    height: 35px;
  }

  .control-card {
    padding: 0.75rem;
  }

  .control-label {
    font-size: 0.875rem;
    margin-bottom: 0.5rem;
  }

  .counter-control {
    gap: 0.5rem;
    margin-bottom: 0.5rem;
  }

  .counter-btn {
    width: 36px;
    height: 36px;
    font-size: 20px;
  }

  .counter-input {
    width: 80px;
    height: 36px;
    font-size: 1.25rem;
  }

  .button-group {
    gap: 0.25rem;
  }

  .btn {
    padding: 8px 12px;
    font-size: 12px;
  }

  .footer-card {
    padding: 0.75rem;
  }

  .footer-content {
    flex-direction: column;
    align-items: stretch;
  }

  .last-updated {
    text-align: center;
    font-size: 0.75rem;
  }

  .refresh-btn {
    width: 100%;
    justify-content: center;
    padding: 10px 16px;
    font-size: 13px;
  }
}
</style>
