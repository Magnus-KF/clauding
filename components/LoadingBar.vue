<script setup lang="ts">
import { ref, computed, onMounted, onUnmounted, PropType } from 'vue'

const props = defineProps({
  speed: {
    type: Number,
    default: 1,
  },
  text: {
    type: [String, Array] as PropType<string | string[]>,
    default: 'Loading',
  },
})

const progress = ref(0)
const spinnerIndex = ref(0)
const currentTextIndex = ref(0)
const textOpacity = ref(1)
const spinnerChars = ['⋯', '⋰', '⋱', '⋮', '∴', '∵', '∷', '⁘', '⁙','◦', '◦◦', '◦◦◦']
const defaultTexts = ["Vibing", "Cerebrating", "Thinking", "Loading", "lOaDInG", "Skulking", "Pondering"]

const displayText = computed(() => {
  if (typeof props.text === 'string') {
    return props.text
  } else if (Array.isArray(props.text) && props.text.length > 0) {
    return props.text[currentTextIndex.value]
  } else {
    return defaultTexts[currentTextIndex.value]
  }
})

const textList = computed(() => {
  if (typeof props.text === 'string') {
    return [props.text]
  } else if (Array.isArray(props.text) && props.text.length > 0) {
    return props.text
  } else {
    return defaultTexts
  }
})

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
  
  // Use raw value to allow going beyond container
  return {
    width: `${progress.value}%`,
    transition: progress.value > 0 ? 'width 1s ease-out, opacity 1s ease-in-out' : 'none',
    opacity,
  }
})

// Animation interval references
let loadingInterval: number | null = null
let spinnerInterval: number | null = null
let textCycleTimeout: number | null = null

const cycleText = () => {
  // Start fade out
  textOpacity.value = 0
  
  // After fade out, change text and start fade in
  setTimeout(() => {
    currentTextIndex.value = (currentTextIndex.value + 1) % textList.value.length
    textOpacity.value = 1
    
    // Schedule next cycle
    textCycleTimeout = window.setTimeout(cycleText, 10000)
  }, 500) // Wait for fade out (500ms)
}

onMounted(() => {
  // Start with a minimal delay to ensure proper rendering
  setTimeout(() => {
    // Animate loading progress
    loadingInterval = window.setInterval(() => {
      progress.value += 0.5
      
      // When reaching max, reset the bar immediately
      if (progress.value >= 300) {
        progress.value = 0
      }
    }, 30)
  }, 100)  // Initial delay to ensure everything is in place
  
  // Animate spinner character
  spinnerInterval = window.setInterval(() => {
    spinnerIndex.value = (spinnerIndex.value + 1) % spinnerChars.length
  }, 100) // Fast animation for smooth transitions
  
  // Only start text cycling if we have multiple texts
  if (textList.value.length > 1) {
    textCycleTimeout = window.setTimeout(cycleText, 10000)
  }
})

onUnmounted(() => {
  // Clear intervals on component unmount
  if (loadingInterval) clearInterval(loadingInterval)
  if (spinnerInterval) clearInterval(spinnerInterval)
  if (textCycleTimeout) clearTimeout(textCycleTimeout)
})
</script>

<template>
  <div class="loading-container">
    <div class="fixed-width-container">
      <div class="loading-text-container">
        <span class="loading-spinner">{{ spinnerChars[spinnerIndex] }}</span>
        <span class="loading-text" :style="{ opacity: textOpacity, transition: 'opacity 0.5s ease-in-out' }">{{ displayText }}</span>
      </div>
      
      <div class="loading-bar-container">
        <!-- Bounding box -->
        <div class="loading-bar-outline"></div>
        
        <!-- Progress bar that extends -->
        <div class="loading-bar-progress" :style="progressStyle"></div>
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
}

.fixed-width-container {
  width: 400px;
  display: flex;
  flex-direction: column;
  align-items: center;
}

.loading-text-container {
  font-size: 2rem;
  font-weight: bold;
  display: flex;
  justify-content: center;
  align-items: center;
  margin-bottom: 1rem;
  width: 100%;
  padding-right: 3rem; /* Shift content slightly to the left */
}

.loading-text {
  color: #d97757;
  margin-left: 0.5rem;
}

.loading-spinner {
  display: inline-block;
  color: #d97757;
  font-size: 1.5rem;
  width: 2.5rem;
  text-align: right;
  margin-right: 0.5rem;
}

.loading-bar-container {
  position: relative;
  width: 100%;
  height: 30px;
}

.loading-bar-outline {
  position: absolute;
  width: 100%;
  height: 100%;
  border: 3px solid currentColor;
  border-radius: 15px;
  box-sizing: border-box;
  z-index: 2;
  pointer-events: none;
}

.loading-bar-progress {
  position: absolute;
  top: 3px; /* Adjust for the border */
  left: 3px; /* Align with the inside of the outline */
  height: calc(100% - 6px); /* Adjust for the border */
  background: linear-gradient(90deg, #4a8cff, #63b3ed);
  border-radius: 12px;
  transition: width 0.3s ease-out;
  z-index: 1;
  max-width: 1200px; /* Allow extending far beyond the container's edge */
  transform-origin: left center;
}
</style>