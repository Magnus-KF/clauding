<script setup lang="ts">
import { ref, computed, onMounted, onUnmounted } from 'vue'

const props = defineProps({
  speed: {
    type: Number,
    default: 1,
  },
  text: {
    type: String,
    default: 'Loading',
  },
})

const progress = ref(0)
const spinnerIndex = ref(0)
const spinnerChars = ['⋯', '⋰', '⋱', '⋮', '∴', '∵', '∷', '⁘', '⁙','◦', '◦◦', '◦◦◦']
// 
// Calculate the position of the progress bar
const progressStyle = computed(() => {
  // Calculate a smooth opacity transition
  let opacity = 0;
  
  if (progress.value > 0 && progress.value <= 50) {
    // Fade in between 10% and 15%
    opacity = (progress.value - 10) / 5;
  } else if (progress.value > 50) {
    // Fully visible after 15%
    opacity = 1;
  }
  
  return {
    width: `${progress.value}%`,
    transition: progress.value > 0 ? 'width 1s ease-out, opacity 1s ease-in-out' : 'none',
    opacity,
    // visibility: progress.value > 0 ? 'visible' : 'hidden',
  }
})

// Animation interval references
let loadingInterval: number | null = null
let spinnerInterval: number | null = null

onMounted(() => {
  // Start with a minimal delay to ensure proper rendering
  setTimeout(() => {
    // Animate loading progress
    loadingInterval = window.setInterval(() => {
      progress.value += 0.5
      
      // When reaching max, reset the bar immediately
      if (progress.value >= 400) {
        progress.value = 0
      }
    }, 30)
  }, 100)  // Initial delay to ensure everything is in place
  
  // Animate spinner character
  spinnerInterval = window.setInterval(() => {
    spinnerIndex.value = (spinnerIndex.value + 1) % spinnerChars.length
  }, 100) // Fast animation for smooth transitions
})

onUnmounted(() => {
  // Clear intervals on component unmount
  if (loadingInterval) clearInterval(loadingInterval)
  if (spinnerInterval) clearInterval(spinnerInterval)
})
</script>

<template>
  <div class="loading-container">
    <div class="loading-text-container">
      <span class="loading-text">{{ text }}</span>
      <span class="loading-spinner">{{ spinnerChars[spinnerIndex] }}</span>
    </div>
    
    <div class="loading-bar-container">
      <!-- Progress bar that extends -->
      <div class="loading-bar-progress" :style="progressStyle"></div>
      
      <!-- Bounding box overlaid on top -->
      <div class="loading-bar-mask">
        <div class="loading-bar-outline"></div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.loading-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  width: 100%;
  gap: 1rem;
}

.loading-text-container {
  font-size: 2rem;
  font-weight: bold;
  display: flex;
  justify-content: center;
  align-items: center;
  margin-bottom: 0.5rem;
  margin-left: 4rem; /* Move text to the right */
}

.loading-text {
  color: #d97757;
  margin-right: 0.5rem;
}

.loading-spinner {
  display: inline-block;
  color: #d97757;
  font-size: 1.5rem;
  min-width: 4rem;
  text-align: left;
}

.loading-bar-container {
  position: relative;
  width: 100%;
  height: 30px;
}

.loading-bar-mask {
  position: absolute;
  width: 100%;
  height: 100%;
  top: 0;
  left: 0;
  pointer-events: none;
  z-index: 2;
}

.loading-bar-outline {
  width: 60%;
  height: 100%;
  border: 3px solid currentColor;
  border-radius: 15px;
  margin: 0 auto;
  box-sizing: border-box;
}

.loading-bar-progress {
  position: absolute;
  top: 3px; /* Adjust for the border */
  left: calc(20% + 3px); /* Align with the inside of the outline */
  height: calc(100% - 6px); /* Adjust for the border */
  background: linear-gradient(90deg, #4a8cff, #63b3ed);
  border-radius: 12px;
  transition: width 0.3s ease-out;
  z-index: 1;
  max-width: 400%; /* Allow extending far beyond the canvas edge */
}
</style>