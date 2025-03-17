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
        @mouseover="handleMouseOver(event)"
        @mouseout="handleMouseOut()"
      >
        <div class="event-marker"></div>
        <div class="event-label" :style="{ opacity: getLabelOpacity(event) }">
          <div class="event-title" v-html="parseEmoji(event.title)"></div>
          <div class="event-date" :style="{ opacity: getDateOpacity(event), height: getDateHeight(event) }">
            {{ formatDate(event.date) }}
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { computed, ref } from 'vue'

interface TimelineEvent {
  date: string;
  title: string;
  slideIndex: number;
}

const hoveredEvent = ref<TimelineEvent | null>(null);

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

// Handle mouse events
const handleMouseOver = (event: TimelineEvent) => {
  hoveredEvent.value = event;
}

const handleMouseOut = () => {
  hoveredEvent.value = null;
}

// Control label visibility based on hover and active state
const getLabelOpacity = (event: TimelineEvent) => {
  if (isActive(event)) return 1;
  if (hoveredEvent.value === event && !hasActiveEventNearby(event)) return 1;
  return 0.7;
}

// Control date visibility based on hover and active state
const getDateOpacity = (event: TimelineEvent) => {
  if (isActive(event)) return 1;
  if (hoveredEvent.value === event && !hasActiveEventNearby(event)) return 0.8;
  return 0;
}

const getDateHeight = (event: TimelineEvent) => {
  if (isActive(event)) return 'auto';
  if (hoveredEvent.value === event && !hasActiveEventNearby(event)) return 'auto';
  return '0';
}

// Check if there's an active event nearby (to prevent hover effects close to active events)
const hasActiveEventNearby = (event: TimelineEvent) => {
  return props.timelineEvents.some(e => 
    isActive(e) && 
    e !== event && 
    Math.abs(calculatePosition(e) - calculatePosition(event)) < 15
  );
}

// Format date for display (e.g., "Jan 15, 2023")
const formatDate = (dateString: string) => {
  const date = new Date(dateString)
  return date.toLocaleDateString('en-US', { 
    month: 'short', 
    day: 'numeric', 
    year: 'numeric' 
  })
}

// Parse emoji shortcodes in text
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
    ':rotating_light:': '🚨'
  }
  
  // Replace all emoji shortcodes with their Unicode equivalents
  let parsedText = text
  Object.entries(emojiMap).forEach(([shortcode, emoji]) => {
    parsedText = parsedText.replace(new RegExp(shortcode, 'g'), emoji)
  })
  
  return parsedText
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
  const minSpacing = 15 // Minimum 15% spacing between events (increased from 8%)
  
  // Calculate index-based position (evenly spaced)
  // Add edge padding to prevent labels from being cut off at the edges
  const edgePadding = 10 // 10% padding from edges
  const indexBasedPercent = totalEvents > 1 
    ? edgePadding + (eventIndex / (totalEvents - 1)) * (100 - 2 * edgePadding)
    : 50 // Center if just one event
    
  // Calculate weight factor based on how stretched the timeline is
  // If events are very unevenly distributed, we'll lean more toward index-based positioning
  const maxRawGap = getMaxRawGap()
  const minRawGap = getMinRawGap() // Get the minimum gap between events
  
  // Calculate blend factor based on both max gaps (too large) and min gaps (too small)
  let blendFactor = maxRawGap > maxGapPercent 
    ? Math.min(1, (maxRawGap - maxGapPercent) / 50) // Gradually blend based on how much the max gap exceeds our limit
    : 0
    
  // If we have very close dates, increase blend factor to favor index-based positioning
  if (minRawGap < 5) { // If any two events are within 5% of timeline span
    blendFactor = Math.max(blendFactor, 0.7 + (5 - minRawGap) / 5 * 0.3) // 0.7 to 1.0 blend factor
  }
  
  // For first and last events, use more index-based positioning to keep them from edges
  let adjustedBlendFactor = blendFactor
  if (eventIndex === 0 || eventIndex === totalEvents - 1) {
    adjustedBlendFactor = Math.max(0.7, blendFactor) // At least 70% index-based for edge events
  }
  
  // Blend between raw percentage and index-based percentage
  let blendedPercentage = (1 - adjustedBlendFactor) * rawPercentage + adjustedBlendFactor * indexBasedPercent
  
  // Force minimum spacing between events for better readability
  // Find closest events and adjust if needed, with stronger minimum spacing enforcement
  if (totalEvents > 1) {
    // First calculate all positions
    const positions = props.timelineEvents.map(e => {
      const eDate = new Date(e.date).getTime()
      const ePercentage = ((eDate - startDate.value) / timelineSpan.value) * 100
      const eIndex = props.timelineEvents.findIndex(evt => evt.date === e.date)
      // Use same edge-aware index-based percentage
      const eIndexBasedPercent = edgePadding + (eIndex / (totalEvents - 1)) * (100 - 2 * edgePadding)
      // Apply same edge-aware blending
      const eAdjustedBlendFactor = (eIndex === 0 || eIndex === totalEvents - 1) 
        ? Math.max(0.7, blendFactor) 
        : blendFactor
      const eBlendedPercentage = (1 - eAdjustedBlendFactor) * ePercentage + eAdjustedBlendFactor * eIndexBasedPercent
      
      return {
        date: e.date,
        originalPosition: eBlendedPercentage,  // Store original position for reference
        position: Math.max(0, Math.min(100, eBlendedPercentage)),
        index: eIndex  // Store the actual event index
      }
    }).sort((a, b) => a.position - b.position)
    
    // Apply minimum spacing constraints in a separate iterative pass
    // This ensures each event can be properly positioned with respect to all others
    let passCount = 0
    let adjustmentMade = true
    
    // Iteratively adjust positions until no more changes are needed or max passes reached
    while (adjustmentMade && passCount < 3) {
      adjustmentMade = false
      passCount++
      
      // Forward pass - push events to the right if too close to previous event
      for (let i = 1; i < positions.length; i++) {
        const prevPos = positions[i-1].position
        const currPos = positions[i].position
        
        if (currPos - prevPos < minSpacing) {
          positions[i].position = prevPos + minSpacing
          adjustmentMade = true
        }
      }
      
      // Backward pass - push events to the left if too close to next event
      for (let i = positions.length - 2; i >= 0; i--) {
        const nextPos = positions[i+1].position
        const currPos = positions[i].position
        
        if (nextPos - currPos < minSpacing) {
          // Avoid pushing below 0 or crowding the previous event
          const minPos = i > 0 ? positions[i-1].position + minSpacing : edgePadding
          positions[i].position = Math.max(minPos, nextPos - minSpacing)
          adjustmentMade = true
        }
      }
    }
    
    // Finally, apply the adjusted position to our current event
    const currentPosition = positions.find(p => p.date === event.date)
    if (currentPosition) {
      blendedPercentage = currentPosition.position
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

// Helper function to find the minimum gap between consecutive events in raw percentage
const getMinRawGap = () => {
  if (!props.timelineEvents || props.timelineEvents.length <= 1) return 100
  
  let minGap = 100 // Start with maximum possible value
  const sortedEvents = [...props.timelineEvents].sort((a, b) => 
    new Date(a.date).getTime() - new Date(b.date).getTime()
  )
  
  for (let i = 1; i < sortedEvents.length; i++) {
    const prevDate = new Date(sortedEvents[i-1].date).getTime()
    const currDate = new Date(sortedEvents[i].date).getTime()
    const gap = ((currDate - prevDate) / timelineSpan.value) * 100
    if (gap > 0) { // Only consider non-zero gaps
      minGap = Math.min(minGap, gap)
    }
  }
  
  return minGap
}

// Calculate the current progress percentage (matching our new spacing algorithm)
const progressPercentage = computed(() => {
  if (!props.currentDate) return 0
  
  const currentDateTimestamp = new Date(props.currentDate).getTime()
  
  // Find the nearest timeline events (before and after the current date)
  const sortedEvents = [...props.timelineEvents].sort((a, b) => 
    new Date(a.date).getTime() - new Date(b.date).getTime()
  )
  
  // Get the actual position of the first event for exact matching
  const firstEventPos = calculatePosition(sortedEvents[0])
  
  // If we're at the first event, align progress exactly with it
  if (props.currentDate === sortedEvents[0]?.date) {
    return firstEventPos
  }
  
  // If we're before the first event, show a small initial progress
  if (currentDateTimestamp <= new Date(sortedEvents[0]?.date || '').getTime()) {
    // Show minimal progress before first event
    return Math.max(2, firstEventPos * 0.5) // At least 2% progress, up to half the first event position
  }
  
  // If we're after the last event
  if (currentDateTimestamp >= new Date(sortedEvents[sortedEvents.length - 1]?.date || '').getTime()) {
    // Use the position of the last event, not quite 100% to match our adjusted edge padding
    const lastEventPos = calculatePosition(sortedEvents[sortedEvents.length - 1])
    return Math.min(100, lastEventPos + 2) // Slightly past the last event position
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
  
  // If this is the exact event that matches the current date
  const exactMatch = event.date === props.currentDate;
  
  // If we're at the first event and the current date is before or equal to it
  const isFirstEventAndBeforeIt = 
    event === props.timelineEvents[0] && 
    new Date(props.currentDate).getTime() <= new Date(event.date).getTime();
  
  // Special handling for the last two timeline events (Monday and Today)
  // This fixes the specific issue where both Monday and Today were showing as active
  if (props.timelineEvents.length >= 2) {
    const lastTwoEvents = [
      props.timelineEvents[props.timelineEvents.length - 2],
      props.timelineEvents[props.timelineEvents.length - 1]
    ];
    
    // If this event is one of the last two events
    if (lastTwoEvents.includes(event)) {
      // Only make it active if it's an exact match with currentDate
      return exactMatch;
    }
  }
  
  // For all other cases, follow the original logic
  return exactMatch || isFirstEventAndBeforeIt;
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
  height: 60px; /* Increased height for better fit */
  padding: 0.5rem 2rem 0.75rem 2rem;
  background: rgba(0, 0, 0, 0.05);
  position: relative;
  z-index: 10;
  box-shadow: 0 2px 5px rgba(0, 0, 0, 0.05);
  transition: background 0.3s ease, box-shadow 0.3s ease;
  /* Allow timeline to be fully visible */
  overflow: visible;
  margin-bottom: 25px; /* Increased margin at bottom to prevent overlap with content */
}

.project-timeline:hover {
  background: rgba(0, 0, 0, 0.07);
  box-shadow: 0 3px 8px rgba(0, 0, 0, 0.08);
}

.timeline-track {
  position: relative;
  height: 4px;
  background: rgba(0, 0, 0, 0.08);
  margin: 24px 0 12px 0; /* Increased top margin to fit labels */
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
  /* Ensure there's enough space on edges */
  min-width: 100px;
}

.timeline-event.active {
  z-index: 30; /* Higher z-index for active events */
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
  transition: all 0.3s ease;
  background: rgba(255, 255, 255, 0.6);
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
  transition: all 0.3s ease;
  color: #3b82f6;
  margin-top: 2px;
}

.timeline-event:hover .event-marker {
  transform: scale(1.2);
}

.timeline-event.active .event-marker {
  background: #3b82f6;
  width: 16px;
  height: 16px;
  margin: -6px auto;
  box-shadow: 0 0 0 4px rgba(59, 130, 246, 0.2);
}

.timeline-event.active .event-label {
  font-weight: bold;
  background: rgba(59, 130, 246, 0.1);
  border: 1px solid rgba(59, 130, 246, 0.2);
  padding: 2px 8px;
  max-width: none; /* Show full width for active events */
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