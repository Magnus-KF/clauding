<template>
  <div class="project-timeline">
    <div class="timeline-track">
      <div class="timeline-progress" :style="{ width: progressPercentage + '%' }"></div>
      
      <div 
        v-for="event in timelineEvents" 
        :key="event.date"
        class="timeline-event"
        :class="{ active: isActive(event), passed: isPassed(event) }"
        :style="{ left: calculatePosition(event) + '%' }"
      >
        <div class="event-marker"></div>
        <div class="event-label">
          <div class="event-title">{{ event.title }}</div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { computed } from 'vue'

interface TimelineEvent {
  date: string;
  title: string;
  slideIndex: number;
}

const props = defineProps({
  currentDate: {
    type: String,
    required: true
  },
  timelineEvents: {
    type: Array as () => TimelineEvent[],
    required: true,
    default: () => []
  },
  currentSlideIndex: {
    type: Number,
    required: true
  }
})

// Format date for display (e.g., "Jan 15, 2023")
const formatDate = (dateString: string) => {
  const date = new Date(dateString)
  return date.toLocaleDateString('en-US', { 
    month: 'short', 
    day: 'numeric', 
    year: 'numeric' 
  })
}

// Calculate timeline start and end dates with safety checks
const startDate = computed(() => {
  if (!props.timelineEvents || props.timelineEvents.length === 0) {
    return new Date().getTime()
  }
  return new Date(props.timelineEvents[0].date).getTime()
})

const endDate = computed(() => {
  if (!props.timelineEvents || props.timelineEvents.length === 0) {
    return new Date().getTime()
  }
  return new Date(props.timelineEvents[props.timelineEvents.length - 1].date).getTime()
})

const timelineSpan = computed(() => {
  const span = endDate.value - startDate.value
  // Ensure we have a non-zero span
  return span > 0 ? span : 1
})

// Calculate the position percentage for an event
const calculatePosition = (event: TimelineEvent) => {
  if (!event || !event.date) return 0
  
  const eventDate = new Date(event.date).getTime()
  const percentage = ((eventDate - startDate.value) / timelineSpan.value) * 100
  return Math.max(0, Math.min(100, percentage))
}

// Calculate the current progress percentage
const progressPercentage = computed(() => {
  if (!props.currentDate) return 0
  
  const currentDateTimestamp = new Date(props.currentDate).getTime()
  const percentage = ((currentDateTimestamp - startDate.value) / timelineSpan.value) * 100
  return Math.max(0, Math.min(100, percentage))
})

// Check if an event is active (current)
const isActive = (event: TimelineEvent) => {
  if (!event || !event.date || !props.currentDate) return false
  return event.date === props.currentDate
}

// Check if an event has passed
const isPassed = (event: TimelineEvent) => {
  if (!event || !event.date || !props.currentDate) return false
  return new Date(event.date) < new Date(props.currentDate)
}
</script>

<style>
.project-timeline {
  width: 100%;
  height: 50px; /* Reduced from 80px */
  padding: 0.75rem 2rem; /* Reduced padding */
  background: rgba(0, 0, 0, 0.05);
  position: relative;
  z-index: 10;
}

.timeline-track {
  position: relative;
  height: 3px;
  background: rgba(0, 0, 0, 0.1);
  margin: 15px 0; /* Reduced from 30px */
}

.timeline-progress {
  position: absolute;
  height: 100%;
  background: #3b82f6;
  transition: width 0.5s ease;
}

.timeline-event {
  position: absolute;
  transform: translateX(-50%);
  z-index: 20;
}

.event-marker {
  width: 12px;
  height: 12px;
  border-radius: 50%;
  background: rgba(0, 0, 0, 0.2);
  border: 2px solid white;
  margin: -5px auto;
  transition: all 0.3s ease;
}

.event-label {
  position: absolute;
  top: 10px; /* Reduced from 15px */
  left: 50%;
  transform: translateX(-50%);
  white-space: nowrap;
  font-size: 0.8rem;
  opacity: 0.7;
  transition: all 0.3s ease;
}

.event-title {
  font-weight: 600;
}

.timeline-event.active .event-marker {
  background: #3b82f6;
  width: 16px;
  height: 16px;
  margin: -7px auto;
}

.timeline-event.active .event-label {
  opacity: 1;
  top: 10px; /* Reduced from 18px */
  font-weight: bold;
}

.timeline-event.passed .event-marker {
  background: #3b82f6;
}
</style>