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
  gridSize: 50,               // Number of cells in simulation grid (fewer for hexagons)
  damping: 0.96,              // Wave damping factor (higher = faster settling)
  
  // Wave parameters
  waveIntensity: 4.0,         // Strength of wave pulse when clicked
  waveRadius: 5,              // Radius of wave pulse
  
  // Lily pad parameters
  numLilies: 12,              // Number of lily pads to generate
  minLilySize: 2,             // Minimum lily size in hexes
  maxLilySize: 5,             // Maximum lily size in hexes
  
  // Colors
  waterColor: '#4a9ddb',      // Deeper blue water color
  lilyColorLight: '#4caf50',  // Light lily pad color (light green)
  lilyColorMid: '#2e7d32',    // Medium lily pad color
  lilyColorDark: '#1b5e20',   // Dark lily pad color
  lilyFlower: '#f06292'       // Optional pink lily flower color
});

// Hexagonal grid helpers
// Cube coordinates for hexagons (axial coordinates converted to cube)
interface HexCoord {
  q: number; // column
  r: number; // row
  s: number; // computed as -q-r
}

// For pixel calculations
interface Point {
  x: number;
  y: number;
}

// Hex directions - six neighbors in hex grid
const hexDirections: HexCoord[] = [
  { q: 1, r: 0, s: -1 },  // East
  { q: 0, r: 1, s: -1 },  // Southeast
  { q: -1, r: 1, s: 0 },  // Southwest
  { q: -1, r: 0, s: 1 },  // West
  { q: 0, r: -1, s: 1 },  // Northwest
  { q: 1, r: -1, s: 0 }   // Northeast
];

// Wave simulation state
let grid: Map<string, number> = new Map(); // Current height using hex coordinates
let lastGrid: Map<string, number> = new Map(); // Previous height
let lilyPads: { 
  center: HexCoord,
  hexes: HexCoord[],
  pixelCenter: Point,
  size: number,
  hasFlower: boolean,
  flowerPosition?: Point
}[] = [];

// Create a map to track which cells are occupied by lily pads
// Keys are string representations of hex coordinates
let lilyPadMap: Set<string> = new Set();

// Helper functions for hex coordinates
const hexToString = (hex: HexCoord): string => {
  return `${hex.q},${hex.r},${hex.s}`;
};

const stringToHex = (str: string): HexCoord => {
  const [q, r, s] = str.split(',').map(Number);
  return { q, r, s };
};

// Hex arithmetic
const hexAdd = (a: HexCoord, b: HexCoord): HexCoord => {
  return { q: a.q + b.q, r: a.r + b.r, s: a.s + b.s };
};

// Get pixel position from hex coordinates
const hexToPixel = (hex: HexCoord, hexSize: number): Point => {
  const x = hexSize * (3/2 * hex.q);
  const y = hexSize * (Math.sqrt(3)/2 * hex.q + Math.sqrt(3) * hex.r);
  return { x, y };
};

// Get hex from pixel coordinates
const pixelToHex = (point: Point, hexSize: number): HexCoord => {
  const q = (2/3 * point.x) / hexSize;
  const r = (-1/3 * point.x + Math.sqrt(3)/3 * point.y) / hexSize;
  const s = -q - r;
  
  // We need to round to the nearest hex
  let rq = Math.round(q);
  let rr = Math.round(r);
  let rs = Math.round(s);
  
  const qDiff = Math.abs(rq - q);
  const rDiff = Math.abs(rr - r);
  const sDiff = Math.abs(rs - s);
  
  // Adjust to ensure q + r + s = 0
  if (qDiff > rDiff && qDiff > sDiff) {
    rq = -rr - rs;
  } else if (rDiff > sDiff) {
    rr = -rq - rs;
  } else {
    rs = -rq - rr;
  }
  
  return { q: rq, r: rr, s: rs };
};

// Get hex neighbors
const hexNeighbors = (hex: HexCoord): HexCoord[] => {
  return hexDirections.map(dir => hexAdd(hex, dir));
};

// Calculate hex distance
const hexDistance = (a: HexCoord, b: HexCoord): number => {
  return (Math.abs(a.q - b.q) + Math.abs(a.r - b.r) + Math.abs(a.s - b.s)) / 2;
};

// Draw a single hexagon at given center point
const drawHexagon = (ctx: CanvasRenderingContext2D, center: Point, size: number): void => {
  ctx.beginPath();
  for (let i = 0; i < 6; i++) {
    const angle = 2 * Math.PI / 6 * i;
    const x = center.x + size * Math.cos(angle);
    const y = center.y + size * Math.sin(angle);
    if (i === 0) {
      ctx.moveTo(x, y);
    } else {
      ctx.lineTo(x, y);
    }
  }
  ctx.closePath();
};

// Initialize the simulation grid with hexagonal coordinates
const initializeGrid = () => {
  grid = new Map();
  lastGrid = new Map();
  lilyPadMap = new Set();
};

// Generate random lily pads using hex grid with improved solid filling
const generateLilyPads = () => {
  lilyPads = [];
  lilyPadMap.clear();
  
  // Calculate hex size to ensure full canvas coverage
  // Use a slightly smaller grid size to ensure hexes are big enough
  const effectiveGridSize = Math.max(config.gridSize * 0.8, 30);
  
  // Ensure the grid covers the full canvas plus a margin
  // Make hexes smaller to fit more on screen
  const hexSize = Math.min(
    canvasWidth.value / (1.6 * effectiveGridSize), 
    canvasHeight.value / (Math.sqrt(3) * effectiveGridSize * 0.7)
  );
  
  // Calculate grid bounds with extra margin to ensure full coverage
  const margin = 4; // Increased margin
  const gridAspectRatio = canvasWidth.value / canvasHeight.value;
  
  // Calculate bounds based on aspect ratio - increased multipliers for wider coverage
  let qMax = Math.ceil(effectiveGridSize * 0.75 * gridAspectRatio) + margin;
  let qMin = -qMax;
  let rMax = Math.ceil(effectiveGridSize * 0.75) + margin;
  let rMin = -rMax;
  
  // Helper function for flood fill algorithm to create solid lily pads
  const floodFillLilyPad = (centerHex: HexCoord, radius: number): HexCoord[] => {
    const result: HexCoord[] = [];
    const visited: Set<string> = new Set();
    const queue: HexCoord[] = [centerHex];
    
    // Add center hex
    const centerKey = hexToString(centerHex);
    visited.add(centerKey);
    result.push(centerHex);
    
    // Process queue
    while (queue.length > 0) {
      const current = queue.shift()!;
      
      // For each neighbor
      for (const direction of hexDirections) {
        const neighbor = hexAdd(current, direction);
        const neighborKey = hexToString(neighbor);
        
        // Check if we've visited this hex before
        if (visited.has(neighborKey)) continue;
        
        // Check if this neighbor is within the lily pad radius
        const distance = hexDistance(centerHex, neighbor);
        if (distance <= radius) {
          visited.add(neighborKey);
          result.push(neighbor);
          queue.push(neighbor);
        }
      }
    }
    
    return result;
  };
  
  // Attempt to create lily pads
  for (let i = 0; i < config.numLilies; i++) {
    // Determine size of this lily pad in hexes
    const size = Math.floor(Math.random() * (config.maxLilySize - config.minLilySize + 1)) + config.minLilySize;
    
    // Find a place for the lily pad that doesn't overlap with existing ones
    // Try up to 20 times to find a valid position
    let validPosition = false;
    let centerHex: HexCoord = { q: 0, r: 0, s: 0 };
    let lilyHexes: HexCoord[] = [];
    
    for (let attempt = 0; attempt < 20 && !validPosition; attempt++) {
      // Find a random center position
      const q = Math.floor(Math.random() * (qMax - qMin - size)) + qMin + Math.floor(size/2);
      const r = Math.floor(Math.random() * (rMax - rMin - size)) + rMin + Math.floor(size/2);
      const s = -q - r;
      centerHex = { q, r, s };
      
      // Use flood fill to create a solid lily pad - this ensures no holes
      lilyHexes = floodFillLilyPad(centerHex, size);
      
      // Check if this position is clear with additional buffer
      validPosition = true;
      
      // First check if all hexes are within bounds
      for (const hex of lilyHexes) {
        if (hex.q < qMin || hex.q > qMax || hex.r < rMin || hex.r > rMax) {
          validPosition = false;
          break;
        }
      }
      
      // Then check for overlap with existing lily pads AND their surrounding area
      if (validPosition) {
        for (const hex of lilyHexes) {
          // Check the hex itself
          if (lilyPadMap.has(hexToString(hex))) {
            validPosition = false;
            break;
          }
          
          // Check all neighboring hexes to ensure lily pads don't touch
          const neighbors = hexNeighbors(hex);
          for (const neighbor of neighbors) {
            if (lilyPadMap.has(hexToString(neighbor))) {
              validPosition = false;
              break;
            }
          }
          
          if (!validPosition) break;
        }
      }
    }
    
    // If we found a valid position, mark these cells as lily pad
    if (validPosition && lilyHexes.length > 0) {
      // Mark cells as lily pad
      for (const hex of lilyHexes) {
        lilyPadMap.add(hexToString(hex));
      }
      
      // Convert center hex to pixel coordinates
      const pixelCenter = hexToPixel(centerHex, hexSize);
      
      // 50% chance to add a flower
      const hasFlower = Math.random() > 0.5;
      let flowerPosition: Point | undefined;
      let flowerHex: HexCoord | undefined;
      
      if (hasFlower && lilyHexes.length > 0) {
        // Find hexes that are closer to the center for flower placement
        const centerHexes = lilyHexes.filter(hex => {
          const distance = hexDistance(centerHex, hex);
          return distance <= Math.min(2, Math.floor(size / 3));
        });
        
        // Pick a random hex from the center area
        const targetHex = centerHexes.length > 0 
          ? centerHexes[Math.floor(Math.random() * centerHexes.length)]
          : centerHex;
        flowerHex = targetHex;
        
        // Convert to pixel coordinates
        const flowerPixelPos = hexToPixel(targetHex, hexSize);
        
        // Add slight random offset within the hex, but ensure it stays in bounds
        const randomOffset = hexSize * 0.2; // Smaller offset to stay within hex
        
        // Position in canvas coordinates
        flowerPosition = {
          x: flowerPixelPos.x + canvasWidth.value / 2 + (Math.random() * 2 - 1) * randomOffset,
          y: flowerPixelPos.y + canvasHeight.value / 2 + (Math.random() * 2 - 1) * randomOffset
        };
      }
      
      // Add the lily pad
      lilyPads.push({
        center: centerHex,
        hexes: lilyHexes,
        pixelCenter: {
          x: pixelCenter.x + canvasWidth.value / 2,  // Center in canvas
          y: pixelCenter.y + canvasHeight.value / 2  // Center in canvas
        },
        size: hexSize,
        hasFlower,
        flowerPosition // Already in canvas coordinates from earlier
      });
    }
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
    config.gridSize = Math.round(50 * aspectRatio);
  } else {
    config.gridSize = 50;
  }
  
  // Reinitialize simulation with new dimensions
  initializeGrid();
  generateLilyPads();
};

// Check if a point is inside a lily pad
const isInsideLilyPad = (x: number, y: number): boolean => {
  // Calculate hex size
  const hexSize = Math.min(
    canvasWidth.value / (2 * config.gridSize), 
    canvasHeight.value / (Math.sqrt(3) * config.gridSize)
  );
  
  // Adjust coordinates to be relative to the hex grid origin (center of canvas)
  const adjustedX = x - canvasWidth.value / 2;
  const adjustedY = y - canvasHeight.value / 2;
  
  // Convert to hex coordinates
  const hex = pixelToHex({ x: adjustedX, y: adjustedY }, hexSize);
  
  // Check if this hex is part of a lily pad
  return lilyPadMap.has(hexToString(hex));
};

// Handle canvas click
const handleClick = (event: MouseEvent) => {
  const canvas = canvasRef.value;
  if (!canvas) return;
  
  const rect = canvas.getBoundingClientRect();
  const x = event.clientX - rect.left;
  const y = event.clientY - rect.top;
  
  // Calculate hex size
  const hexSize = Math.min(
    canvasWidth.value / (2 * config.gridSize), 
    canvasHeight.value / (Math.sqrt(3) * config.gridSize)
  );
  
  // Adjust coordinates to be relative to the hex grid origin (center of canvas)
  const adjustedX = x - canvasWidth.value / 2;
  const adjustedY = y - canvasHeight.value / 2;
  
  // Convert to hex coordinates
  const clickedHex = pixelToHex({ x: adjustedX, y: adjustedY }, hexSize);
  
  // Create a wave pulse in hex grid
  const radius = config.waveRadius;
  for (let dq = -radius; dq <= radius; dq++) {
    for (let dr = Math.max(-radius, -dq-radius); dr <= Math.min(radius, -dq+radius); dr++) {
      const ds = -dq - dr;
      const hex = { q: clickedHex.q + dq, r: clickedHex.r + dr, s: clickedHex.s + ds };
      const hexKey = hexToString(hex);
      
      // Check distance
      if (hexDistance(clickedHex, hex) <= radius) {
        // Check if hex is not part of a lily pad
        if (!lilyPadMap.has(hexKey)) {
          // Create wave pulse
          grid.set(hexKey, config.waveIntensity);
        }
      }
    }
  }
};

// Update the simulation state using hex grid
const updateSimulation = () => {
  // Create a new grid for the next state
  const newGrid: Map<string, number> = new Map();
  
  // Process all water hexes - using a more standard wave equation approach
  // Similar to the classic square grid water simulation but adapted for hex
  
  // First, get all active hexes and neighbors we need to evaluate
  const activeHexes: Set<string> = new Set();
  
  // Get all current active hexes
  for (const [hexKey, height] of grid.entries()) {
    if (Math.abs(height) > 0.01) {
      activeHexes.add(hexKey);
      
      // Add all neighbors as well
      const hex = stringToHex(hexKey);
      const neighbors = hexNeighbors(hex);
      for (const neighbor of neighbors) {
        if (!lilyPadMap.has(hexToString(neighbor))) {
          activeHexes.add(hexToString(neighbor));
        }
      }
    }
  }
  
  // Update all active hexes based on wave equation
  for (const hexKey of activeHexes) {
    // Skip lily pad cells
    if (lilyPadMap.has(hexKey)) continue;
    
    const hex = stringToHex(hexKey);
    const neighbors = hexNeighbors(hex);
    
    // Sum of all neighboring heights
    let neighborSum = 0;
    let validNeighborCount = 0;
    
    for (const neighbor of neighbors) {
      const neighborKey = hexToString(neighbor);
      
      // Skip lily pad neighbors
      if (lilyPadMap.has(neighborKey)) continue;
      
      // Get neighboring height (or 0 if not present)
      neighborSum += grid.get(neighborKey) || 0;
      validNeighborCount++;
    }
    
    // Get current and last height for this cell
    const currentHeight = grid.get(hexKey) || 0;
    const lastHeight = lastGrid.get(hexKey) || 0;
    
    // Use the wave equation: next = (2 * current - last + c * (average of neighbors - current)) * damping
    // Where c is a constant that controls the wave speed (usually 2)
    const averageNeighbor = validNeighborCount > 0 ? neighborSum / validNeighborCount : 0;
    const nextHeight = (
      2 * currentHeight - 
      lastHeight + 
      1.5 * (averageNeighbor - currentHeight)
    ) * config.damping;
    
    // Set new height
    newGrid.set(hexKey, nextHeight);
  }
  
  // Update grid state
  lastGrid = grid;
  grid = newGrid;
};

// Render the simulation to canvas using hex grid
const renderCanvas = () => {
  const canvas = canvasRef.value;
  if (!canvas) return;
  
  const ctx = canvas.getContext('2d');
  if (!ctx) return;
  
  // Clear canvas with a base water color
  ctx.fillStyle = config.waterColor;
  ctx.fillRect(0, 0, canvasWidth.value, canvasHeight.value);
  
  // Calculate hex size
  const hexSize = Math.min(
    canvasWidth.value / (2 * config.gridSize), 
    canvasHeight.value / (Math.sqrt(3) * config.gridSize)
  );
  
  // Draw water with waves
  ctx.save();
  
  // Draw water waves directly on the main canvas for reliability
  // No temporary canvas to avoid potential width/height issues
  
  // Calculate larger grid bounds to ensure full canvas coverage
  const aspectRatio = canvasWidth.value / canvasHeight.value;
  // Increase size to ensure coverage of entire canvas - using larger multipliers
  const qMax = Math.ceil(config.gridSize * 0.9 * aspectRatio) + 8; // Extra margin for sides
  const qMin = -qMax;
  const rMax = Math.ceil(config.gridSize * 0.9) + 4;
  const rMin = -rMax;
  
  // Draw a subtle background wave pattern
  for (let q = qMin; q <= qMax; q++) {
    for (let r = rMin; r <= rMax; r++) {
      const s = -q - r;
      const hex = { q, r, s };
      const hexKey = hexToString(hex);
      
      // Skip lily pad cells
      if (lilyPadMap.has(hexKey)) continue;
      
      // Convert to pixel coordinates (centered in canvas)
      const pixelPos = hexToPixel(hex, hexSize);
      const centerX = pixelPos.x + canvasWidth.value / 2;
      const centerY = pixelPos.y + canvasHeight.value / 2;
      
      // Only draw water cells that are visible on the canvas (plus a small margin)
      const margin = hexSize * 2;
      if (centerX < -margin || centerX > canvasWidth.value + margin || 
          centerY < -margin || centerY > canvasHeight.value + margin) {
        continue;
      }
      
      // Get the current hex height
      const height = grid.get(hexKey) || 0;
      
      // Draw basic water for all cells
      const baseWaterAlpha = 0.1;
      ctx.fillStyle = `rgba(120, 180, 235, ${baseWaterAlpha})`;
      drawHexagon(ctx, {x: centerX, y: centerY}, hexSize * 0.98);
      ctx.fill();
      
      // Add wave effect for cells with movement
      if (Math.abs(height) > 0.01) {
        // Calculate color based on height
        // Limit height range for more consistent colors
        const clampedHeight = Math.max(-2, Math.min(2, height));
        
        // Positive heights are lighter blues (waves)
        // Negative heights are darker blues (troughs)
        const baseR = 100;
        const baseG = 160;
        const baseB = 235;
        
        const r = baseR + clampedHeight * 25;
        const g = baseG + clampedHeight * 25;
        const b = baseB + clampedHeight * 15;
        const a = Math.min(0.7, 0.2 + Math.abs(clampedHeight) * 0.25);
        
        ctx.fillStyle = `rgba(${r}, ${g}, ${b}, ${a})`;
        drawHexagon(ctx, {x: centerX, y: centerY}, hexSize * 0.95);
        ctx.fill();
      }
    }
  }
  
  ctx.restore();
  
  // Draw lily pads
  ctx.save();
  
  // For each lily pad, draw with simple organic shading
  for (const lilyPad of lilyPads) {
    // Draw each hex in the lily pad
    for (const hex of lilyPad.hexes) {
      // Convert to pixel coordinates
      const pixelPos = hexToPixel(hex, hexSize);
      const x = pixelPos.x + canvasWidth.value / 2;
      const y = pixelPos.y + canvasHeight.value / 2;
      
      // Draw the hex
      ctx.beginPath();
      for (let i = 0; i < 6; i++) {
        const angle = 2 * Math.PI / 6 * i;
        const pointX = x + hexSize * 0.98 * Math.cos(angle); // Slightly smaller for visual appeal
        const pointY = y + hexSize * 0.98 * Math.sin(angle);
        if (i === 0) {
          ctx.moveTo(pointX, pointY);
        } else {
          ctx.lineTo(pointX, pointY);
        }
      }
      ctx.closePath();
      
      // Calculate distance from center of lily pad (for color variation)
      const distance = hexDistance(hex, lilyPad.center);
      const maxDistance = lilyPad.size;
      const normalizedDistance = distance / maxDistance;
      
      // Use different green shades based on position
      // Create random organic patches within the lily pad
      const randomFactor = Math.sin(hex.q * 5) * Math.cos(hex.r * 3) * 0.5 + 0.5; // Pseudo-random but stable
      
      // Apply color based on distance and random factor for organic appearance
      if (normalizedDistance > 0.7 || randomFactor > 0.7) {
        // Edges and some random patches are darker green
        ctx.fillStyle = config.lilyColorDark;
      } else if (normalizedDistance < 0.3 && randomFactor < 0.3) {
        // Center and some random patches are lighter green
        ctx.fillStyle = config.lilyColorLight;
      } else {
        // Most areas are medium green
        ctx.fillStyle = config.lilyColorMid;
      }
      
      // Fill with green lily pad color
      ctx.fill();
      
      // Add simple edge outline for definition if this is an edge hex
      // Check if all neighbors are part of this lily pad
      const isEdge = hexNeighbors(hex).some(neighbor => 
        !lilyPad.hexes.some(padHex => 
          padHex.q === neighbor.q && padHex.r === neighbor.r
        )
      );
      
      if (isEdge) {
        ctx.strokeStyle = 'rgba(15, 89, 18, 0.4)';
        ctx.lineWidth = 0.5;
        ctx.stroke();
      }
    }
    
    // Draw lily flower if this pad has one
    if (lilyPad.hasFlower && lilyPad.flowerPosition) {
      const {x, y} = lilyPad.flowerPosition;
      
      // Draw flower petals
      ctx.fillStyle = config.lilyFlower;
      
      // Draw flower as a small cluster of circles
      const petalSize = hexSize * 0.35;
      for (let i = 0; i < 5; i++) {
        const angle = (i / 5) * Math.PI * 2;
        const petalX = x + Math.cos(angle) * petalSize * 0.3;
        const petalY = y + Math.sin(angle) * petalSize * 0.3;
        
        ctx.beginPath();
        ctx.arc(petalX, petalY, petalSize * 0.5, 0, Math.PI * 2);
        ctx.fill();
      }
      
      // Draw center of flower
      ctx.fillStyle = '#fff59d'; // Yellow center
      ctx.beginPath();
      ctx.arc(x, y, petalSize * 0.3, 0, Math.PI * 2);
      ctx.fill();
    }
  }
  
  ctx.restore();
};

// Helper to check if a hex is at the edge of a lily pad
const isLilyPadEdge = (hex: HexCoord): boolean => {
  // Check all six neighbors
  const neighbors = hexNeighbors(hex);
  
  // If any neighbor is not part of a lily pad, this is an edge
  for (const neighbor of neighbors) {
    if (!lilyPadMap.has(hexToString(neighbor))) {
      return true;
    }
  }
  
  return false;
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
  generateLilyPads();
  
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