<script setup lang="ts">
import { computed } from 'vue'

interface File {
  name: string;
  extension?: string;
}

const props = defineProps({
  title: {
    type: String,
    required: true,
  },
  author: {
    type: String,
    required: true,
  },
  message: {
    type: String,
    default: '',
  },
  changes: {
    type: String,
    default: '',
  },
  files: {
    type: Array,
    default: () => [],
  },
  date: {
    type: String,
    // default: new Date().toLocaleDateString(),
  },
});

// Process files to extract extension for syntax highlighting
const processedFiles = computed(() => {
  return props.files.map((file: string) => {
    const parts = file.split('.');
    return {
      name: file,
      extension: parts.length > 1 ? parts[parts.length - 1] : null
    };
  });
});

// Get corresponding color for file extension
const getExtensionColor = (extension: string | null | undefined) => {
  if (!extension) return 'text-gray-400';
  
  const colorMap: Record<string, string> = {
    py: 'text-blue-500',
    js: 'text-yellow-500',
    ts: 'text-blue-400',
    vue: 'text-green-500',
    html: 'text-orange-500',
    css: 'text-purple-500',
    md: 'text-gray-500',
    json: 'text-yellow-400',
    txt: 'text-gray-400',
  };
  
  return colorMap[extension] || 'text-gray-400';
};
</script>

<template>
  <div class="commit-info w-full max-w-3xl mx-auto my-2" border="~ blue-400 rounded" overflow="hidden" style="max-height: 80vh; overflow-y: auto;" scale="95%">
    <!-- Header section with author and date -->
    <div class="commit-header flex justify-between p-2" bg="blue-400 opacity-80" text="white">
      <div class="author flex items-center gap-2">
        <div i-carbon-user text="xl"></div>
        <span font="medium">{{ author }}</span>
      </div>
      <div class="date">
        {{ date }}
      </div>
    </div>
    
    <!-- Title section -->
    <div class="commit-title p-3" bg="light-900 dark:dark-800">
      <h2 font="bold" text="lg" mb="1">{{ title }}</h2>
      
      <!-- Message block -->
      <div v-if="message" class="commit-message my-2 p-2" bg="light-800 dark:dark-700" border="l-4 blue-400" text="left sm">
        <p>{{ message }}</p>
      </div>
      
      <!-- Changes summary -->
      <div v-if="changes" class="commit-changes mt-2 mb-1" text="xs blue-500">
        <div flex="~ gap-1" items="center">
          <div i-carbon-data-change></div>
          <span>{{ changes }}</span>
        </div>
      </div>
      
      <!-- Files list -->
      <div v-if="files.length" class="commit-files mt-2 p-2" bg="light-800 dark:dark-700" border="~ rounded" text="xs">
        <div font="medium" mb="1" flex="~ gap-1" items="center">
          <div i-carbon-document></div>
          <span>Modified files</span>
        </div>
        <ul class="file-list space-y-0.5 pl-4">
          <li v-for="file in processedFiles" :key="file.name" flex="~ gap-1" items="center">
            <div i-carbon-document-blank text="2xs"></div>
            <span :class="getExtensionColor(file.extension)">{{ file.name }}</span>
          </li>
        </ul>
      </div>
    </div>
  </div>
</template>