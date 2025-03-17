<script setup lang="ts">
const props = defineProps({
  metrics: {
    type: Array,
    default: () => [],
  },
  height: {
    type: String,
    default: '300px'
  },
  width: {
    type: String,
    default: '100%'
  },
  showLabels: {
    type: Boolean,
    default: false
  }
});

// Static data to show in the visualization
const locChange = "-28%";
const coverageChange = "+70%";
const complexityChange = "-29%";
</script>

<template>
  <div class="code-evolution-container" 
       :style="{ height: props.height, width: props.width }">
    
    <!-- Simplified static visualization -->
    <div class="viz-container">
      <div class="data-row">
        <div class="label">Lines of Code</div>
        <div class="value green">{{ locChange }}</div>
        <div class="bar-container">
          <div class="bar green" style="width: 72%"></div>
        </div>
      </div>
      
      <div class="data-row">
        <div class="label">Test Coverage</div>
        <div class="value blue">{{ coverageChange }}</div>
        <div class="bar-container">
          <div class="bar blue" style="width: 85%"></div>
        </div>
      </div>
      
      <div class="data-row">
        <div class="label">Complexity</div>
        <div class="value pink">{{ complexityChange }}</div>
        <div class="bar-container">
          <div class="bar pink" style="width: 71%"></div>
        </div>
      </div>
    </div>
    
    <!-- Years timeline -->
    <div class="timeline">
      <div class="year">2021</div>
      <div class="year">2022</div>
      <div class="year">2023</div>
      <div class="year">2024</div>
      <div class="year">2025</div>
    </div>
  </div>
</template>

<style>
.code-evolution-container {
  position: relative;
  overflow: hidden;
  border-radius: 8px;
  background: rgba(30, 30, 30, 0.2);
  border: 1px solid rgba(255, 255, 255, 0.1);
  padding: 20px;
  box-sizing: border-box;
  display: flex;
  flex-direction: column;
}

.viz-container {
  flex: 1;
  display: flex;
  flex-direction: column;
  justify-content: space-around;
}

.data-row {
  display: flex;
  align-items: center;
  margin-bottom: 20px;
}

.label {
  width: 120px;
  font-size: 14px;
  color: rgba(255, 255, 255, 0.8);
}

.value {
  width: 70px;
  font-size: 20px;
  font-weight: bold;
  text-align: right;
  margin-right: 15px;
}

.green {
  color: rgba(65, 184, 131, 1);
}

.blue {
  color: rgba(97, 175, 239, 1);
}

.pink {
  color: rgba(237, 100, 166, 1);
}

.bar-container {
  flex: 1;
  height: 12px;
  background: rgba(255, 255, 255, 0.1);
  border-radius: 6px;
  overflow: hidden;
}

.bar {
  height: 100%;
  border-radius: 6px;
}

.bar.green {
  background: linear-gradient(to right, rgba(65, 184, 131, 0.3), rgba(65, 184, 131, 0.8));
}

.bar.blue {
  background: linear-gradient(to right, rgba(97, 175, 239, 0.3), rgba(97, 175, 239, 0.8));
}

.bar.pink {
  background: linear-gradient(to right, rgba(237, 100, 166, 0.3), rgba(237, 100, 166, 0.8));
}

.timeline {
  display: flex;
  justify-content: space-between;
  margin-top: 20px;
  padding: 0 10px;
}

.year {
  font-size: 12px;
  color: rgba(255, 255, 255, 0.6);
}
</style>