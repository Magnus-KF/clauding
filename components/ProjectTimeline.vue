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
          <div class="event-date">{{ formatDate(event.date) }}</div>
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

// Calculate the position percentage for an event with max distance limiting
const calculatePosition = (event: TimelineEvent) => {
  if (!event || !event.date) return 0
  
  // Original date-based calculation
  const eventDate = new Date(event.date).getTime()
  const rawPercentage = ((eventDate - startDate.value) / timelineSpan.value) * 100
  
  // Get the index of this event for index-based spacing
  const eventIndex = props.timelineEvents.findIndex(e => e.date === event.date)
  const totalEvents = props.timelineEvents.length
  
  if (eventIndex === -1 || totalEvents <= 1) {
    return Math.max(0, Math.min(100, rawPercentage))
  }
  
  // Blend between date-based and index-based positioning
  // maxGapPercent defines the maximum percentage gap allowed between consecutive events
  const maxGapPercent = 25 // Maximum 25% gap between any two events
  const minSpacing = 8 // Minimum 8% spacing between events
  
  // Calculate index-based position (evenly spaced)
  const indexBasedPercent = totalEvents > 1 
    ? (eventIndex / (totalEvents - 1)) * 100 
    : 0
    
  // Calculate weight factor based on how stretched the timeline is
  // If events are very unevenly distributed, we'll lean more toward index-based positioning
  const maxRawGap = getMaxRawGap()
  const blendFactor = maxRawGap > maxGapPercent 
    ? Math.min(1, (maxRawGap - maxGapPercent) / 50) // Gradually blend based on how much the max gap exceeds our limit
    : 0
  
  // Blend between raw percentage and index-based percentage
  let blendedPercentage = (1 - blendFactor) * rawPercentage + blendFactor * indexBasedPercent
  
  // Force minimum spacing between events for better readability
  // Find closest events and adjust if needed
  if (totalEvents > 1) {
    const positions = props.timelineEvents.map(e => {
      const eDate = new Date(e.date).getTime()
      const ePercentage = ((eDate - startDate.value) / timelineSpan.value) * 100
      const eIndex = props.timelineEvents.findIndex(evt => evt.date === e.date)
      const eBlendedPercentage = (1 - blendFactor) * ePercentage + blendFactor * (eIndex / (totalEvents - 1)) * 100
      return {
        date: e.date,
        position: Math.max(0, Math.min(100, eBlendedPercentage))
      }
    }).sort((a, b) => a.position - b.position)
    
    const currentPosition = positions.find(p => p.date === event.date)
    if (currentPosition) {
      const currentIndex = positions.indexOf(currentPosition)
      if (currentIndex > 0) {
        const prevPosition = positions[currentIndex - 1].position
        if (currentPosition.position - prevPosition < minSpacing) {
          blendedPercentage = prevPosition + minSpacing
        }
      }
      if (currentIndex < positions.length - 1) {
        const nextPosition = positions[currentIndex + 1].position
        if (nextPosition - currentPosition.position < minSpacing) {
          // Only adjust if we're not already pushing against a preceding event
          if (currentIndex === 0 || (currentPosition.position - positions[currentIndex - 1].position >= minSpacing)) {
            blendedPercentage = nextPosition - minSpacing
          }
        }
      }
    }
  }
  
  return Math.max(0, Math.min(100, blendedPercentage))
}

// Helper function to find the maximum gap between consecutive events in raw percentage
const getMaxRawGap = () => {
  if (!props.timelineEvents || props.timelineEvents.length <= 1) return 0
  
  let maxGap = 0
  const sortedEvents = [...props.timelineEvents].sort((a, b) => 
    new Date(a.date).getTime() - new Date(b.date).getTime()
  )
  
  for (let i = 1; i < sortedEvents.length; i++) {
    const prevDate = new Date(sortedEvents[i-1].date).getTime()
    const currDate = new Date(sortedEvents[i].date).getTime()
    const gap = ((currDate - prevDate) / timelineSpan.value) * 100
    maxGap = Math.max(maxGap, gap)
  }
  
  return maxGap
}

// Calculate the current progress percentage (matching our new spacing algorithm)
const progressPercentage = computed(() => {
  if (!props.currentDate) return 0
  
  const currentDateTimestamp = new Date(props.currentDate).getTime()
  
  // Find the nearest timeline events (before and after the current date)
  const sortedEvents = [...props.timelineEvents].sort((a, b) => 
    new Date(a.date).getTime() - new Date(b.date).getTime()
  )
  
  // If we're before the first event
  if (currentDateTimestamp <= new Date(sortedEvents[0]?.date || '').getTime()) {
    return 0
  }
  
  // If we're after the last event
  if (currentDateTimestamp >= new Date(sortedEvents[sortedEvents.length - 1]?.date || '').getTime()) {
    return 100
  }
  
  // Find the events before and after the current date
  let beforeEvent = null
  let afterEvent = null
  
  for (let i = 0; i < sortedEvents.length; i++) {
    const eventDate = new Date(sortedEvents[i].date).getTime()
    if (eventDate > currentDateTimestamp) {
      afterEvent = sortedEvents[i]
      beforeEvent = sortedEvents[i - 1]
      break
    }
  }
  
  if (!beforeEvent || !afterEvent) {
    // Fallback to simple calculation if we can't find surrounding events
    const percentage = ((currentDateTimestamp - startDate.value) / timelineSpan.value) * 100
    return Math.max(0, Math.min(100, percentage))
  }
  
  // Get the positions of the surrounding events
  const beforePos = calculatePosition(beforeEvent)
  const afterPos = calculatePosition(afterEvent)
  
  // Calculate position based on time between the two events
  const beforeTime = new Date(beforeEvent.date).getTime()
  const afterTime = new Date(afterEvent.date).getTime()
  const timeRange = afterTime - beforeTime
  
  if (timeRange <= 0) {
    return beforePos
  }
  
  const timeProgress = (currentDateTimestamp - beforeTime) / timeRange
  const position = beforePos + timeProgress * (afterPos - beforePos)
  
  return Math.max(0, Math.min(100, position))
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
  height: 50px;
  padding: 0.75rem 2rem;
  background: rgba(0, 0, 0, 0.05);
  position: relative;
  z-index: 10;
  box-shadow: 0 2px 5px rgba(0, 0, 0, 0.05);
  transition: background 0.3s ease, box-shadow 0.3s ease;
}

.project-timeline:hover {
  background: rgba(0, 0, 0, 0.07);
  box-shadow: 0 3px 8px rgba(0, 0, 0, 0.08);
}

.timeline-track {
  position: relative;
  height: 4px;
  background: rgba(0, 0, 0, 0.08);
  margin: 15px 0;
  border-radius: 2px;
}

.timeline-progress {
  position: absolute;
  height: 100%;
  background: linear-gradient(90deg, #3b82f6, #60a5fa);
  transition: width 0.5s ease;
  border-radius: 2px;
  box-shadow: 0 0 6px rgba(59, 130, 246, 0.5);
  overflow: hidden;
}

.timeline-progress::after {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: linear-gradient(
    90deg,
    rgba(255, 255, 255, 0) 0%,
    rgba(255, 255, 255, 0.2) 50%,
    rgba(255, 255, 255, 0) 100%
  );
  z-index: 2;
  transform: translateX(-100%);
  animation: progressShine 2.5s infinite;
}

@keyframes progressShine {
  0% {
    transform: translateX(-100%);
  }
  50%, 100% {
    transform: translateX(100%);
  }
}

.timeline-event {
  position: absolute;
  transform: translateX(-50%);
  z-index: 20;
  cursor: pointer;
}

.event-marker {
  width: 12px;
  height: 12px;
  border-radius: 50%;
  background: rgba(0, 0, 0, 0.15);
  border: 2px solid white;
  margin: -4px auto;
  transition: all 0.3s ease;
  box-shadow: 0 1px 3px rgba(0, 0, 0, 0.1);
}

.event-label {
  position: absolute;
  top: 11px;
  left: 50%;
  transform: translateX(-50%);
  white-space: nowrap;
  font-size: 0.75rem;
  opacity: 0.7;
  transition: all 0.3s ease;
  background: rgba(255, 255, 255, 0.8);
  padding: 2px 6px;
  border-radius: 4px;
  max-width: 120px;
  overflow: hidden;
  text-overflow: ellipsis;
  display: flex;
  flex-direction: column;
  align-items: center;
}

.event-title {
  font-weight: 600;
}

.event-date {
  font-size: 0.7rem;
  opacity: 0;
  height: 0;
  transition: all 0.3s ease;
  color: #3b82f6;
}

.timeline-event:hover .event-date {
  opacity: 0.8;
  height: auto;
  margin-top: 2px;
}

.timeline-event.active .event-date {
  opacity: 1;
  height: auto;
  margin-top: 2px;
}

.timeline-event:hover .event-marker {
  transform: scale(1.2);
}

.timeline-event:hover .event-label {
  opacity: 1;
  max-width: none;
}

.timeline-event.active .event-marker {
  background: #3b82f6;
  width: 16px;
  height: 16px;
  margin: -6px auto;
  box-shadow: 0 0 0 4px rgba(59, 130, 246, 0.2);
}

.timeline-event.active .event-label {
  opacity: 1;
  font-weight: bold;
  background: rgba(59, 130, 246, 0.1);
  border: 1px solid rgba(59, 130, 246, 0.2);
  padding: 2px 8px;
}

.timeline-event.passed .event-marker {
  background: #3b82f6;
}

/* Add connecting lines between events */
.timeline-track:before {
  content: '';
  position: absolute;
  width: 100%;
  height: 1px;
  background: repeating-linear-gradient(90deg, transparent, transparent 5px, rgba(0, 0, 0, 0.05) 5px, rgba(0, 0, 0, 0.05) 10px);
  top: 50%;
  transform: translateY(-50%);
  z-index: 1;
}
</style>