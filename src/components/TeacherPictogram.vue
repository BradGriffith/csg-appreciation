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
          <div class="stat-number">
            {{ supportPercentage }}%
          </div>
          <div class="stat-label">
            of Faculty & Staff supported
          </div>
          <div class="stat-sublabel">
            {{ vouchersPurchased }} of {{ goalVouchers }} vouchers purchased
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
      <div class="control-card">
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
    </div>
  </div>
</template>

<script setup>
import { computed, ref } from 'vue'
import TeacherIcon from './TeacherIcon.vue'

const totalIcons = 100
const goalVouchers = 220
const vouchersPurchased = ref(110) // Start at 50%
const iconSize = 40

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
}
</style>
