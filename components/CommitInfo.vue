<script setup lang="ts">
import { computed } from 'vue'
import MarkdownIt from 'markdown-it'

// Initialize the markdown parser
const md = new MarkdownIt({
  html: true,       // Enable HTML tags in source
  breaks: true,     // Convert '\n' in paragraphs into <br>
  linkify: true     // Autoconvert URL-like text to links
})

interface SimpleFile {
  name: string;
  extension?: string;
}

interface DetailedFile {
  path: string;
  status: 'Added' | 'Modified' | 'Deleted' | 'Changed' | string;
  name?: string;
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

// Parse emoji shortcodes, convert Markdown, and handle newlines
const parseEmoji = (text: string) => {
  if (!text) return text
  
  // Map of common emoji shortcodes to unicode emoji
  const emojiMap: Record<string, string> = {
    ':tada:': '🎉',
    ':rocket:': '🚀',
    ':fire:': '🔥',
    ':sparkles:': '✨',
    ':bug:': '🐛',
    ':lock:': '🔒',
    ':bookmark:': '🔖',
    ':zap:': '⚡',
    ':ambulance:': '🚑',
    ':art:': '🎨',
    ':package:': '📦',
    ':memo:': '📝',
    ':bulb:': '💡',
    ':construction:': '🚧',
    ':recycle:': '♻️',
    ':white_check_mark:': '✅',
    ':wrench:': '🔧',
    ':hammer:': '🔨',
    ':ok_hand:': '👌',
    ':truck:': '🚚',
    ':gear:': '⚙️',
    ':books:': '📚',
    ':speech_balloon:': '💬',
    ':globe_with_meridians:': '🌐',
    ':pencil2:': '✏️',
    ':heavy_plus_sign:': '➕',
    ':heavy_minus_sign:': '➖',
    ':rotating_light:': '🚨',
    ':see_no_evil:': '🙈'
  }
  
  // Replace all emoji shortcodes with their Unicode equivalents
  let parsedText = text
  Object.entries(emojiMap).forEach(([shortcode, emoji]) => {
    parsedText = parsedText.replace(new RegExp(shortcode, 'g'), emoji)
  })
  
  // Convert literal '\n' sequences to actual newlines for proper Markdown processing
  parsedText = parsedText.replace(/\\n/g, '\n')
  
  // Use markdown-it to render the text with Markdown formatting
  return md.render(parsedText)
};

// Process files to extract extension for syntax highlighting
const processedFiles = computed(() => {
  return props.files.map((file: any) => {
    // Handle both formats: simple string array and detailed object array
    if (typeof file === 'string') {
      // Simple format - just a file path string
      const parts = file.split('.');
      return {
        name: file,
        path: file,
        extension: parts.length > 1 ? parts[parts.length - 1] : null,
        status: 'Modified' // Default status for backward compatibility
      };
    } else {
      // Detailed format with path and status
      const filePath = file.path;
      const parts = filePath.split('.');
      // Extract the file name from the path
      const pathParts = filePath.split('/');
      const fileName = pathParts[pathParts.length - 1];
      
      return {
        name: fileName,
        path: filePath,
        extension: parts.length > 1 ? parts[parts.length - 1] : null,
        status: file.status || 'Modified'
      };
    }
  });
});

// Get corresponding color for file extension
const getExtensionColor = (extension: string | null | undefined) => {
  if (!extension) return 'github-gray';
  
  const colorMap: Record<string, string> = {
    py: 'github-blue',
    js: 'github-yellow',
    ts: 'github-blue',
    vue: 'github-green',
    html: 'github-orange',
    css: 'github-purple',
    md: 'github-gray',
    json: 'github-yellow',
    txt: 'github-gray',
  };
  
  return colorMap[extension] || 'github-gray';
};
</script>

<template>
  <div class="commit-info w-full max-w-3xl mx-auto my-2" border="~ #1e4273 rounded" overflow="hidden" style="max-height: none; overflow-y: visible; overflow-x: hidden; background-color: #0d1117;" scale="100%">
    <!-- Header section with author and date -->
    <div class="commit-header flex justify-between p-2" style="background-color: #1e4273;" text="white">
      <div class="author flex items-center gap-2">
        <div i-carbon-user text="xl"></div>
        <span font="medium">{{ author }}</span>
      </div>
      <div class="date">
        {{ date }}
      </div>
    </div>
    
    <!-- Title section -->
    <div class="commit-title p-3" style="background-color: #0d1117; color: #ffffff;">
      <h2 font="bold" text="lg" mb="1" v-html="parseEmoji(title)"></h2>
      
      <!-- Message block with Markdown support -->
      <div v-if="message" class="commit-message my-2 p-2" style="background-color: #161b22; color: #e6edf3;" border="l-4 #1e4273" text="left sm">
        <div class="markdown-content" v-html="parseEmoji(message)"></div>
      </div>
      
      <!-- Changes summary -->
      <div v-if="changes" class="commit-changes mt-2 mb-1" text="xs" style="color: #58a6ff;">
        <div flex="~ gap-1" items="center">
          <div i-carbon-data-change></div>
          <span v-html="parseEmoji(changes)"></span>
        </div>
      </div>
      
      <!-- Files list -->
      <div v-if="files.length" class="commit-files mt-2 p-2" style="background-color: #161b22; color: #e6edf3;" border="~ rounded #1e4273" text="xs">
        <div font="medium" mb="2" flex="~ gap-1" items="center">
          <div i-carbon-document></div>
          <span>Changed files</span>
        </div>
        <ul class="file-list space-y-0.5 pl-4">
          <li v-for="file in processedFiles" :key="file.path" flex="~ gap-2" items="center">
            <!-- File icon based on status -->
            <div v-if="file.status === 'Added'" i-carbon-add-alt text="2xs" class="status-icon status-added"></div>
            <div v-else-if="file.status === 'Deleted'" i-carbon-subtract-alt text="2xs" class="status-icon status-deleted"></div>
            <div v-else-if="file.status === 'Modified'" i-carbon-edit text="2xs" class="status-icon status-modified"></div>
            <div v-else-if="file.status === 'Changed'" i-carbon-update-now text="2xs" class="status-icon status-changed"></div>
            <div v-else i-carbon-document-blank text="2xs"></div>
            
            <!-- File name with appropriate styling based on status -->
            <span 
              :class="[getExtensionColor(file.extension), 
                      'file-name', 
                      {'file-deleted': file.status === 'Deleted'}]"
            >
              {{ file.name }}
              <span v-if="file.path !== file.name" class="file-path">{{ file.path.replace(file.name, '') }}</span>
            </span>
          </li>
        </ul>
      </div>
    </div>
  </div>
</template>

<style>
/* Markdown content styling */
.markdown-content {
  font-size: 0.9rem;
}

.markdown-content p {
  margin-bottom: 0.5rem;
}

.markdown-content ul, .markdown-content ol {
  padding-left: 1.5rem;
  margin-bottom: 0.5rem;
}

.markdown-content li {
  margin-bottom: 0.25rem;
}

.markdown-content code {
  background-color: #2d3748;
  padding: 0.1rem 0.2rem;
  border-radius: 0.2rem;
  font-family: monospace;
  font-size: 0.85em;
  color: #e6edf3;
}

.markdown-content pre {
  background-color: #2d3748;
  padding: 0.5rem;
  border-radius: 0.3rem;
  overflow-x: auto;
  margin-bottom: 0.5rem;
}

.markdown-content pre code {
  background-color: transparent;
  padding: 0;
}

.markdown-content a {
  color: #58a6ff;
  text-decoration: none;
}

.markdown-content a:hover {
  text-decoration: underline;
}

.markdown-content blockquote {
  border-left: 3px solid #1e4273;
  padding-left: 0.5rem;
  margin-left: 0.5rem;
  color: #8b949e;
}

.file-name {
  color: #e6edf3;
}

/* GitHub-like file extension colors */
.github-blue {
  color: #58a6ff !important;
}

.github-yellow {
  color: #f1e05a !important;
}

.github-green {
  color: #3fb950 !important;
}

.github-orange {
  color: #e34c26 !important;
}

.github-purple {
  color: #a371f7 !important;
}

.github-gray {
  color: #8b949e !important;
}

/* Status colors and icons */
.status-icon {
  flex-shrink: 0;
}

.status-added {
  color: #3fb950;
}

.status-deleted {
  color: #f85149;
}

.status-modified {
  color: #58a6ff;
}

.status-changed {
  color: #d29922;
}

.file-deleted {
  text-decoration: line-through;
  opacity: 0.7;
}

.file-path {
  opacity: 0.6;
  font-size: 0.85em;
  margin-left: 0.25rem;
}
</style>