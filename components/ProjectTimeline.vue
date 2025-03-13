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
          <div class="event-date">{{ formatDate(event.date) }}</div>
          <div class="event-title">{{ event.title }}</div>
        </div>
      </div>
    </div>
    
    <div class="current-date">
      {{ formatDate(currentDate) }}
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
    required: true
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

// Calculate timeline start and end dates
const startDate = computed(() => new Date(props.timelineEvents[0].date).getTime())
const endDate = computed(() => new Date(props.timelineEvents[props.timelineEvents.length - 1].date).getTime())
const timelineSpan = computed(() => endDate.value - startDate.value)

// Calculate the position percentage for an event
const calculatePosition = (event: TimelineEvent) => {
  const eventDate = new Date(event.date).getTime()
  const percentage = ((eventDate - startDate.value) / timelineSpan.value) * 100
  return percentage
}

// Calculate the current progress percentage
const progressPercentage = computed(() => {
  const currentDateTimestamp = new Date(props.currentDate).getTime()
  const percentage = ((currentDateTimestamp - startDate.value) / timelineSpan.value) * 100
  return Math.max(0, Math.min(100, percentage))
})

// Check if an event is active (current)
const isActive = (event: TimelineEvent) => {
  return event.date === props.currentDate
}

// Check if an event has passed
const isPassed = (event: TimelineEvent) => {
  return new Date(event.date) < new Date(props.currentDate)
}
</script>

<style>
.project-timeline {
  width: 100%;
  height: 80px;
  padding: 1rem 2rem;
  background: rgba(0, 0, 0, 0.05);
  position: relative;
  z-index: 10;
}

.timeline-track {
  position: relative;
  height: 3px;
  background: rgba(0, 0, 0, 0.1);
  margin: 30px 0;
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
  top: 15px;
  left: 50%;
  transform: translateX(-50%);
  white-space: nowrap;
  font-size: 0.8rem;
  opacity: 0.7;
  transition: all 0.3s ease;
}

.event-date {
  font-size: 0.7rem;
  opacity: 0.8;
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
  top: 18px;
  font-weight: bold;
}

.timeline-event.passed .event-marker {
  background: #3b82f6;
}

.current-date {
  position: absolute;
  right: 2rem;
  top: 10px;
  font-size: 0.9rem;
  font-weight: 600;
  color: #3b82f6;
}
</style>