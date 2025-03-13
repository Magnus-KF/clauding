<template>
  <div class="slidev-layout timeline-layout">
    <ProjectTimeline 
      :currentDate="computedCurrentDate" 
      :timelineEvents="timelineEvents"
      :currentSlideIndex="nav?.currentPage || 1" 
    />
    <div class="content">
      <slot />
    </div>
  </div>
</template>

<script setup>
import { computed, inject } from 'vue'
import ProjectTimeline from '../components/ProjectTimeline.vue'

// Access Slidev's nav context
const nav = inject('nav')

// Define props with default values
const props = defineProps({
  currentDate: {
    type: String,
    default: null
  }
})

// These would come from your global state or config
const timelineEvents = [
  { date: '2023-01-15', title: 'Project Kickoff', slideIndex: 2 },
  { date: '2023-03-10', title: 'First Milestone', slideIndex: 5 },
  { date: '2023-05-20', title: 'Major Challenge', slideIndex: 8 },
  { date: '2023-07-15', title: 'Mid-Project Review', slideIndex: 12 },
  { date: '2023-10-05', title: 'Final Testing', slideIndex: 15 },
  { date: '2023-11-30', title: 'Project Delivery', slideIndex: 18 }
]

// Compute the current date based on the slide if not explicitly provided
const computedCurrentDate = computed(() => {
  if (props.currentDate) return props.currentDate
  
  // Find the nearest event for the current slide
  const currentSlide = nav?.currentPage || 1
  const nearestEvent = timelineEvents
    .sort((a, b) => {
      const aDiff = Math.abs(a.slideIndex - currentSlide)
      const bDiff = Math.abs(b.slideIndex - currentSlide)
      return aDiff - bDiff
    })[0]
  
  return nearestEvent?.date || timelineEvents[0].date
})
</script>

<style>
.timeline-layout {
  display: flex;
  flex-direction: column;
  padding: 0;
  height: 100%;
}

.timeline-layout .content {
  padding: 2rem;
  flex: 1;
}
</style>