<script setup lang="ts">
import { ref, reactive, onMounted, onUnmounted } from 'vue';

// Canvas and animation references
const canvasRef = ref<HTMLCanvasElement | null>(null);
const containerRef = ref<HTMLDivElement | null>(null);
const canvasWidth = ref(800);
const canvasHeight = ref(600);
const animationFrameId = ref<number | null>(null);

// Configuration parameters (exposed for easy tweaking)
const config = reactive({
  // Simulation parameters
  gridSize: 100,              // Number of cells in simulation grid
  damping: 0.96,              // Wave damping factor (higher = faster settling)
  
  // Wave parameters
  waveIntensity: 4.0,         // Strength of wave pulse when clicked
  waveRadius: 6,              // Radius of wave pulse
  
  // Rock parameters
  numRocks: 10,               // Number of rocks to generate
  minRockSize: 15,            // Minimum rock size
  maxRockSize: 30,            // Maximum rock size
  useSquareRocks: true,       // Whether to use square rocks instead of circles
  useTriangles: false,        // Whether to use triangle rocks
  
  // Colors
  waterColor: '#e0f7fa',      // Base water color
  rockColorLight: '#e67e22',  // Light rock color
  rockColorDark: '#d35400',   // Dark rock color
  rockOutline: '#a04000'      // Rock outline color
});

// Wave simulation state
let grid: number[][] = []; // Current height
let lastGrid: number[][] = []; // Previous height
let rocks: { x: number, y: number, size: number, type: 'square' | 'triangle', rotation?: number }[] = [];

// Initialize the simulation grid
const initializeGrid = () => {
  grid = Array(config.gridSize).fill(0).map(() => Array(config.gridSize).fill(0));
  lastGrid = Array(config.gridSize).fill(0).map(() => Array(config.gridSize).fill(0));
};

// Generate random rocks
const generateRocks = () => {
  rocks = [];
  for (let i = 0; i < config.numRocks; i++) {
    const size = Math.random() * (config.maxRockSize - config.minRockSize) + config.minRockSize;
    const x = Math.random() * canvasWidth.value;
    const y = Math.random() * canvasHeight.value;
    
    // Randomly choose between square and triangle if both are enabled
    let type: 'square' | 'triangle' = 'square';
    if (config.useSquareRocks && config.useTriangles) {
      type = Math.random() > 0.5 ? 'square' : 'triangle';
    } else if (config.useTriangles) {
      type = 'triangle';
    } else {
      type = 'square';
    }
    
    // Add rotation for triangles
    const rotation = type === 'triangle' ? Math.random() * Math.PI * 2 : 0;
    
    rocks.push({ x, y, size, type, rotation });
  }
};

// Resize canvas to fit container
const resizeCanvas = () => {
  if (!containerRef.value) return;
  
  const { width, height } = containerRef.value.getBoundingClientRect();
  canvasWidth.value = width;
  canvasHeight.value = height;
  
  if (canvasRef.value) {
    canvasRef.value.width = width;
    canvasRef.value.height = height;
  }
  
  // Adjust grid size based on screen dimensions
  const aspectRatio = width / height;
  if (aspectRatio > 1) {
    config.gridSize = Math.round(100 * aspectRatio);
  } else {
    config.gridSize = 100;
  }
  
  // Reinitialize simulation with new dimensions
  initializeGrid();
  generateRocks();
};

// Check if a point is inside a rock
const isInsideRock = (x: number, y: number): boolean => {
  for (const rock of rocks) {
    const dx = x - rock.x;
    const dy = y - rock.y;
    
    if (rock.type === 'square') {
      // For squares, use a rectangular bounds check
      const halfSize = rock.size / 2;
      if (Math.abs(dx) < halfSize && Math.abs(dy) < halfSize) {
        return true;
      }
    } else if (rock.type === 'triangle') {
      // For triangles, use a simplified polygon check
      // This is an approximation that creates a circular boundary
      const distanceSquared = dx * dx + dy * dy;
      if (distanceSquared < rock.size * rock.size / 2) {
        return true;
      }
    }
  }
  return false;
};

// Handle canvas click
const handleClick = (event: MouseEvent) => {
  const canvas = canvasRef.value;
  if (!canvas) return;
  
  const rect = canvas.getBoundingClientRect();
  const x = event.clientX - rect.left;
  const y = event.clientY - rect.top;
  
  // Map click position to grid
  const gridX = Math.floor(x / canvasWidth.value * config.gridSize);
  const gridY = Math.floor(y / canvasHeight.value * config.gridSize);
  
  // Create a wave pulse
  const radius = config.waveRadius;
  for (let i = Math.max(0, gridX - radius); i < Math.min(config.gridSize, gridX + radius); i++) {
    for (let j = Math.max(0, gridY - radius); j < Math.min(config.gridSize, gridY + radius); j++) {
      const dx = i - gridX;
      const dy = j - gridY;
      const distanceSquared = dx * dx + dy * dy;
      if (distanceSquared < radius * radius) {
        // Check if click is not on a rock
        const pixelX = i * canvasWidth.value / config.gridSize;
        const pixelY = j * canvasHeight.value / config.gridSize;
        if (!isInsideRock(pixelX, pixelY)) {
          grid[i][j] = config.waveIntensity; // Create wave pulse with configured intensity
        }
      }
    }
  }
};

// Update the simulation state
const updateSimulation = () => {
  // Create a new grid for the next state
  const newGrid: number[][] = Array(config.gridSize).fill(0).map(() => Array(config.gridSize).fill(0));
  
  // Update each cell based on neighbors
  for (let i = 1; i < config.gridSize - 1; i++) {
    for (let j = 1; j < config.gridSize - 1; j++) {
      const pixelX = i * canvasWidth.value / config.gridSize;
      const pixelY = j * canvasHeight.value / config.gridSize;
      
      // Skip update for cells inside rocks
      if (isInsideRock(pixelX, pixelY)) {
        newGrid[i][j] = 0;
        continue;
      }
      
      // Calculate new height based on neighboring cells
      const val = (
        grid[i-1][j] + 
        grid[i+1][j] + 
        grid[i][j-1] + 
        grid[i][j+1]
      ) / 2 - lastGrid[i][j];
      
      // Apply damping
      newGrid[i][j] = val * config.damping;
    }
  }
  
  // Update grid state
  lastGrid = grid;
  grid = newGrid;
};

// Render the simulation to canvas
const renderCanvas = () => {
  const canvas = canvasRef.value;
  if (!canvas) return;
  
  const ctx = canvas.getContext('2d');
  if (!ctx) return;
  
  // Clear canvas with a base water color
  ctx.fillStyle = config.waterColor;
  ctx.fillRect(0, 0, canvasWidth.value, canvasHeight.value);
  
  // Draw water
  const cellWidth = canvasWidth.value / config.gridSize;
  const cellHeight = canvasHeight.value / config.gridSize;
  
  ctx.save();
  for (let i = 0; i < config.gridSize; i++) {
    for (let j = 0; j < config.gridSize; j++) {
      const height = grid[i][j];
      
      if (Math.abs(height) < 0.05) continue; // Skip nearly flat water
      
      // Calculate color based on height
      const r = 120 + height * 15;
      const g = 180 + height * 15;
      const b = 235 + height * 10;
      const a = Math.min(0.7, Math.abs(height) * 0.5 + 0.2);
      
      ctx.fillStyle = `rgba(${r}, ${g}, ${b}, ${a})`;
      ctx.fillRect(
        i * cellWidth, 
        j * cellHeight, 
        cellWidth + 1, 
        cellHeight + 1
      );
    }
  }
  ctx.restore();
  
  // Draw rocks with a geometric appearance
  for (const rock of rocks) {
    // Draw shadow
    ctx.save();
    if (rock.type === 'square') {
      // Draw square shadow
      ctx.fillStyle = 'rgba(0, 0, 0, 0.2)';
      ctx.fillRect(
        rock.x - rock.size/2 + 2, 
        rock.y - rock.size/2 + 2, 
        rock.size, 
        rock.size
      );
      
      // Draw square rock
      const gradient = ctx.createLinearGradient(
        rock.x - rock.size/2,
        rock.y - rock.size/2,
        rock.x + rock.size/2,
        rock.y + rock.size/2
      );
      
      gradient.addColorStop(0, config.rockColorLight);
      gradient.addColorStop(1, config.rockColorDark);
      
      ctx.fillStyle = gradient;
      ctx.fillRect(
        rock.x - rock.size/2,
        rock.y - rock.size/2,
        rock.size,
        rock.size
      );
      
      ctx.strokeStyle = config.rockOutline;
      ctx.lineWidth = 1;
      ctx.strokeRect(
        rock.x - rock.size/2,
        rock.y - rock.size/2,
        rock.size,
        rock.size
      );
    } else if (rock.type === 'triangle') {
      // Apply rotation for triangle
      ctx.translate(rock.x, rock.y);
      if (rock.rotation) {
        ctx.rotate(rock.rotation);
      }
      
      // Draw triangle shadow
      ctx.fillStyle = 'rgba(0, 0, 0, 0.2)';
      ctx.beginPath();
      ctx.moveTo(2, -rock.size/2 + 2);
      ctx.lineTo(rock.size/2 + 2, rock.size/2 + 2);
      ctx.lineTo(-rock.size/2 + 2, rock.size/2 + 2);
      ctx.closePath();
      ctx.fill();
      
      // Draw triangle rock
      const gradient = ctx.createLinearGradient(
        0, -rock.size/2,
        0, rock.size/2
      );
      
      gradient.addColorStop(0, config.rockColorLight);
      gradient.addColorStop(1, config.rockColorDark);
      
      ctx.fillStyle = gradient;
      ctx.beginPath();
      ctx.moveTo(0, -rock.size/2);
      ctx.lineTo(rock.size/2, rock.size/2);
      ctx.lineTo(-rock.size/2, rock.size/2);
      ctx.closePath();
      ctx.fill();
      
      ctx.strokeStyle = config.rockOutline;
      ctx.lineWidth = 1;
      ctx.stroke();
    }
    ctx.restore();
  }
};

// Animation loop
const animate = () => {
  updateSimulation();
  renderCanvas();
  animationFrameId.value = requestAnimationFrame(animate);
};

// Initialize the component
onMounted(() => {
  // Set up resize handler
  window.addEventListener('resize', resizeCanvas);
  
  // Initial setup
  resizeCanvas();
  initializeGrid();
  generateRocks();
  
  // Start animation
  animate();
});

// Clean up
onUnmounted(() => {
  window.removeEventListener('resize', resizeCanvas);
  
  if (animationFrameId.value !== null) {
    cancelAnimationFrame(animationFrameId.value);
  }
});
</script>

<template>
  <div class="zen-garden-container" ref="containerRef">
    <canvas 
      ref="canvasRef" 
      @click="handleClick"
      class="zen-garden-canvas"
    ></canvas>
    
    <!-- Simple control panel -->
    <div class="control-panel" v-if="false"> <!-- Disabled by default, enable if needed -->
      <div class="control-group">
        <label>Wave Intensity</label>
        <input type="range" v-model="config.waveIntensity" min="1" max="8" step="0.5" />
        <span>{{ config.waveIntensity }}</span>
      </div>
      
      <div class="control-group">
        <label>Wave Radius</label>
        <input type="range" v-model="config.waveRadius" min="3" max="10" step="1" />
        <span>{{ config.waveRadius }}</span>
      </div>
      
      <div class="control-group">
        <label>Damping</label>
        <input type="range" v-model="config.damping" min="0.9" max="0.99" step="0.01" />
        <span>{{ config.damping }}</span>
      </div>
      
      <div class="control-group">
        <label>Rock Style</label>
        <select v-model="config.useSquareRocks">
          <option :value="true">Squares</option>
          <option :value="false">Triangles</option>
        </select>
      </div>
    </div>
  </div>
</template>

<style>
.zen-garden-container {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100vh;
  display: flex;
  justify-content: center;
  align-items: center;
  overflow: hidden;
  background-color: #e0f7fa;
}

.zen-garden-canvas {
  width: 100%;
  height: 100%;
  cursor: pointer;
}

.control-panel {
  position: absolute;
  bottom: 20px;
  right: 20px;
  background: rgba(255, 255, 255, 0.8);
  border-radius: 8px;
  padding: 15px;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
  z-index: 10;
}

.control-group {
  margin-bottom: 12px;
  display: flex;
  align-items: center;
  justify-content: space-between;
}

.control-group label {
  margin-right: 10px;
  min-width: 120px;
  font-size: 14px;
}

.control-group input, 
.control-group select {
  flex: 1;
  margin-right: 10px;
}
</style>