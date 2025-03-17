<template>
  <div class="slidev-layout timeline-layout">
    <ProjectTimeline 
      v-if="timelineEvents.length > 0"
      :currentDate="computedCurrentDate" 
      :timelineEvents="timelineEvents"
      :currentSlideIndex="$nav?.currentPage || 1" 
    />
    <div class="no-timeline-message" v-else>
      <p>No timeline events found. Add slides with 'currentDate' in frontmatter.</p>
    </div>
    <div class="content">
      <slot />
    </div>
  </div>
</template>

<script setup>
import { computed, onMounted, ref } from 'vue'
import ProjectTimeline from '../components/ProjectTimeline.vue'

// We won't use inject('nav') since it's not available
// Instead we'll use the global $nav from Slidev

// Define props with default values
const props = defineProps({
  currentDate: {
    type: String,
    default: null
  },
  // New prop to mark a slide as a timeline event
  timelineEvent: {
    type: Boolean,
    default: false
  },
  // New prop to provide a title for the timeline event (optional)
  eventTitle: {
    type: String,
    default: null
  }
})

// Create a reactive array to store timeline events
const timelineEvents = ref([])

// TIMELINE SYSTEM OVERVIEW:
// This layout implements a timeline system where:
//
// 1. Timeline events are defined by date in chronological order
// 2. Each event gets mapped to a slide position based on its position in the array
// 3. Slides with 'layout: timeline' can provide a 'currentDate' in frontmatter to:
//    - Explicitly specify a date for that slide
//    - Allow the slide to associate with a specific timeline event
// 4. Slides without 'currentDate' will be associated with a timeline event based on their slide number
//
// The timeline below is ordered chronologically, which lets us maintain the order
// without hardcoding slide numbers (which would break when inserting new slides)
const timelineData = [
  { date: '2021-11-05', title: 'git init' },
  { date: '2022-02-09', title: 'Team Seid (8 mo. BMFC)' },
  { date: '2022-10-05', title: '0 BMFC' },
  // { date: '2022-03-10', title: 'First Milestone' },
  { date: '2023-06-27', title: 'FM API' },
  { date: '2024-01-01', title: '2024' },
  { date: '2025-01-01', title: '2025' },
  // { date: '2024-11-30', title: 'Project Delivery' },
  // { date: '2024-12-10', title: 'Key Project Commits' },
  { date: '2025-03-19', title: 'Today' }
];

// Initialize the data, sorting by date to ensure chronological order
timelineEvents.value = timelineData
  .sort((a, b) => new Date(a.date) - new Date(b.date))
  .map((event, index) => ({ ...event, slideIndex: index + 1 }))

// Compute the current date based on several strategies
const computedCurrentDate = computed(() => {
  // Priority 1: If currentDate prop is explicitly provided in this slide, use it
  if (props.currentDate) {
    return props.currentDate
  }
  
  // Priority 2: Use the closest timeline event date based on the current slide
  // Get current slide index (starting from 1)
  const currentSlide = $nav?.currentPage || 1
  
  // Get dates of all timeline events
  const datePoints = timelineEvents.value.map(e => e.date)
  
  // If we have a slide number that's within our timeline events range
  // Select the appropriate date for this slide number
  // The formula maps slide numbers to dates, even if we have irregular intervals
  if (currentSlide >= 1 && currentSlide <= timelineEvents.value.length) {
    // Just get the date corresponding to this slide position
    return datePoints[currentSlide - 1]
  }
  
  // Priority 3: Handle the slide being outside our timeline range
  // Find the closest date (most likely a slide after our last event)
  // Default to the last event date
  return datePoints[datePoints.length - 1] || '2023-01-01'
})
</script>

<style>
.timeline-layout {
  display: flex;
  flex-direction: column;
  padding: 0;
  height: 100%;
  /* Enable vertical scrolling when content overflows */
  overflow-y: auto;
  overflow-x: hidden;
}

.timeline-layout .content {
  padding: 1.5rem;
  padding-top: 0.5rem; /* Reduced top padding since timeline has more space now */
  flex: 1;
  /* Allow content to grow as needed with scroll in parent */
  min-height: min-content;
  height: auto;
  overflow: visible;
}

.no-timeline-message {
  background: rgba(0, 0, 0, 0.05);
  padding: 0.5rem;
  text-align: center;
  font-size: 0.8rem;
  color: #666;
  height: 30px;
  display: flex;
  align-items: center;
  justify-content: center;
}
</style>