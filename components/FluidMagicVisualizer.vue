<script setup lang="ts">
import { ref, computed } from 'vue'

const currentState = ref('volume')
const states = ['volume', 'molar', 'mass']
const stateColors = {
  volume: '#4dabf7',  // blue for liquid volume
  molar: '#82c91e',   // green for molecular
  mass: '#e67700'     // orange for mass
}

const transitions = {
  volume: ['molar'],
  molar: ['volume', 'mass', 'molar'],
  mass: ['molar', 'volume']
}

// Interactive element - allows users to trigger conversions
function convert(to: string) {
  currentState.value = to
}

const containerStyle = computed(() => {
  return {
    backgroundColor: stateColors[currentState.value],
    transition: 'background-color 0.5s, transform 0.5s'
  }
})

const availableConversions = computed(() => {
  return transitions[currentState.value] || []
})

// Simulated fluid data based on current state
const fluidData = computed(() => {
  switch (currentState.value) {
    case 'volume':
      return '100 L'
    case 'molar':
      return '4.2 mol'
    case 'mass':
      return '76 kg'
    default:
      return ''
  }
})
</script>

<template>
  <div class="fluid-magic-container">
    <div class="fluid-vessel" :style="containerStyle">
      <div class="fluid-content">
        <div class="bubbles">
          <div v-for="i in 5" :key="i" class="bubble" :style="{ animationDelay: `${i * 0.5}s` }"></div>
        </div>
        <div class="state-label">{{ currentState.toUpperCase() }}</div>
        <div class="data-value">{{ fluidData }}</div>
      </div>
    </div>
    
    <div class="controls">
      <div class="operations-label">FluidMagic™ Operations</div>
      <div class="button-container">
        <button 
          v-for="state in availableConversions" 
          :key="state"
          @click="convert(state)"
          class="convert-button"
          :style="{ backgroundColor: stateColors[state] }"
        >
          Convert to {{ state }}
        </button>
      </div>
    </div>
  </div>
</template>

<style scoped>
.fluid-magic-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  width: 100%;
  max-width: 600px;
  margin: 0 auto;
  gap: 2rem;
}

.fluid-vessel {
  width: 200px;
  height: 250px;
  border-radius: 10px 10px 5px 5px;
  position: relative;
  overflow: hidden;
  box-shadow: 0 8px 16px rgba(0,0,0,0.2);
  display: flex;
  justify-content: center;
  align-items: center;
}

.fluid-content {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  width: 100%;
  height: 100%;
  color: white;
  text-shadow: 0 1px 2px rgba(0,0,0,0.3);
  position: relative;
  z-index: 2;
}

.state-label {
  font-size: 1.5rem;
  font-weight: bold;
  letter-spacing: 1px;
  margin-bottom: 0.5rem;
}

.data-value {
  font-size: 2rem;
  font-weight: bold;
}

.controls {
  display: flex;
  flex-direction: column;
  align-items: center;
  width: 100%;
  gap: 1rem;
}

.operations-label {
  font-size: 1.2rem;
  font-weight: bold;
  margin-bottom: 0.5rem;
}

.button-container {
  display: flex;
  flex-wrap: wrap;
  justify-content: center;
  gap: 0.75rem;
}

.convert-button {
  padding: 0.5rem 1rem;
  border: none;
  border-radius: 4px;
  color: white;
  font-weight: bold;
  cursor: pointer;
  transition: transform 0.2s, opacity 0.2s;
  box-shadow: 0 2px 4px rgba(0,0,0,0.2);
}

.convert-button:hover {
  transform: translateY(-2px);
}

.convert-button:active {
  transform: translateY(0);
}

.bubbles {
  position: absolute;
  width: 100%;
  height: 100%;
  z-index: 1;
}

.bubble {
  position: absolute;
  bottom: -20px;
  width: 20px;
  height: 20px;
  border-radius: 50%;
  background: rgba(255, 255, 255, 0.3);
  animation: bubble 8s infinite ease-in;
}

.bubble:nth-child(1) { left: 20%; animation-duration: 6s; }
.bubble:nth-child(2) { left: 40%; animation-duration: 8s; animation-delay: 1s; }
.bubble:nth-child(3) { left: 60%; animation-duration: 10s; animation-delay: 2s; }
.bubble:nth-child(4) { left: 80%; animation-duration: 7s; animation-delay: 3s; }
.bubble:nth-child(5) { left: 30%; animation-duration: 9s; animation-delay: 4s; }

@keyframes bubble {
  0% { 
    bottom: -20px;
    opacity: 0.5;
    transform: scale(0.8);
  }
  100% { 
    bottom: 120%;
    opacity: 0;
    transform: scale(1.2);
  }
}
</style>