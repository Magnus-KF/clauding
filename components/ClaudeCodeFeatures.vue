<script setup lang="ts">
defineProps({
  title: {
    type: String,
    default: 'Claude Code Features'
  },
  features: {
    type: Array as () => string[],
    default: () => [
      'Interactive CLI for software engineering tasks',
      'Powerful search capabilities across codebases',
      'Intelligent code generation and editing',
      'Context-aware suggestions and solutions',
      'Remembers preferences and code conventions'
    ]
  }
})
</script>

<template>
  <div class="claude-code-container">
    <h2 class="title">{{ title }}</h2>
    <div class="terminal">
      <div class="terminal-header">
        <div class="terminal-buttons">
          <div class="terminal-button red"></div>
          <div class="terminal-button yellow"></div>
          <div class="terminal-button green"></div>
        </div>
        <div class="terminal-title">claude code</div>
      </div>
      <div class="terminal-content">
        <div class="prompt">$ claude</div>
        <div class="features">
          <ul>
            <li v-for="(feature, index) in features" :key="index" class="feature-item">
              <span class="feature-bullet">→</span> {{ feature }}
            </li>
          </ul>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.claude-code-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  width: 100%;
  height: 100%;
  padding: 1rem;
}

.title {
  font-size: 2.5rem;
  font-weight: 700;
  margin-bottom: 2rem;
  color: #333;
}

.terminal {
  width: 90%;
  max-width: 800px;
  background-color: #1a1a1a;
  border-radius: 8px;
  overflow: hidden;
  box-shadow: 0 10px 20px rgba(0, 0, 0, 0.2);
}

.terminal-header {
  display: flex;
  align-items: center;
  padding: 0.5rem 1rem;
  background-color: #2d2d2d;
  border-bottom: 1px solid #444;
}

.terminal-buttons {
  display: flex;
  gap: 6px;
}

.terminal-button {
  width: 12px;
  height: 12px;
  border-radius: 50%;
}

.red { background-color: #ff5f56; }
.yellow { background-color: #ffbd2e; }
.green { background-color: #27c93f; }

.terminal-title {
  margin-left: auto;
  margin-right: auto;
  font-family: monospace;
  font-size: 14px;
  color: #ddd;
}

.terminal-content {
  padding: 1rem;
  font-family: monospace;
  color: #ddd;
}

.prompt {
  color: #64dd17;
  margin-bottom: 1rem;
}

.features {
  margin-left: 1rem;
}

.feature-item {
  margin-bottom: 1rem;
  display: flex;
  align-items: flex-start;
}

.feature-bullet {
  color: #64dd17;
  margin-right: 0.5rem;
  font-weight: bold;
}
</style>