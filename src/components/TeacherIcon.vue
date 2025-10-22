<template>
  <svg
    :width="size"
    :height="size"
    viewBox="4 0 16 24"
    fill="none"
    xmlns="http://www.w3.org/2000/svg"
    class="person-icon"
  >
    <defs>
      <!-- Define gradient for partial fill from bottom to top -->
      <linearGradient :id="gradientId" x1="0%" y1="100%" x2="0%" y2="0%">
        <stop :offset="`${fillPercentage}%`" :stop-color="fillColor" />
        <stop :offset="`${fillPercentage}%`" stop-color="none" stop-opacity="0" />
      </linearGradient>

      <!-- Clip path for the person shape -->
      <clipPath :id="clipId">
        <!-- Head -->
        <circle cx="12" cy="7" r="3.5" />
        <!-- Body -->
        <path d="M12 12C8 12 5 14 5 16.5V20C5 20.5 5.5 21 6 21H18C18.5 21 19 20.5 19 20V16.5C19 14 16 12 12 12Z" />
      </clipPath>
    </defs>

    <!-- Fill layer with gradient clipped to person shape -->
    <g :clip-path="`url(#${clipId})`">
      <rect
        x="0"
        y="0"
        width="24"
        height="24"
        :fill="`url(#${gradientId})`"
      />
    </g>

    <!-- Head outline -->
    <circle
      cx="12"
      cy="7"
      r="3.5"
      :stroke="strokeColor"
      fill="none"
      stroke-width="1.5"
      stroke-linecap="round"
      stroke-linejoin="round"
    />

    <!-- Body outline -->
    <path
      d="M12 12C8 12 5 14 5 16.5V20C5 20.5 5.5 21 6 21H18C18.5 21 19 20.5 19 20V16.5C19 14 16 12 12 12Z"
      :stroke="strokeColor"
      fill="none"
      stroke-width="1.5"
      stroke-linecap="round"
      stroke-linejoin="round"
    />
  </svg>
</template>

<script setup>
import { computed } from 'vue'

const props = defineProps({
  fillPercentage: {
    type: Number,
    default: 0, // 0-100
    validator: (value) => value >= 0 && value <= 100
  },
  size: {
    type: Number,
    default: 40
  },
  fillColor: {
    type: String,
    default: '#3b82f6'
  },
  strokeColor: {
    type: String,
    default: '#1e40af'
  }
})

// Generate unique IDs for gradient and clip path to avoid conflicts with multiple icons
const gradientId = computed(() => `gradient-${Math.random().toString(36).substr(2, 9)}`)
const clipId = computed(() => `clip-${Math.random().toString(36).substr(2, 9)}`)
</script>

<style scoped>
.person-icon {
  transition: all 0.3s ease;
}

.person-icon:hover {
  transform: scale(1.05);
}
</style>
