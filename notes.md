 // Example of how to use this in your slides.md file:
  // ---
  // layout: TimelineLayout
  // currentDate: '2023-01-15'
  // ---
  
  # Project Kickoff
  January 15, 2023
  
  - Initial team formation
  - Requirements gathering
  - Setting project goals
  
  ---
  // ---
  // layout: TimelineLayout
  // currentDate: '2023-03-10'
  // ---
  
  # First Milestone Reached
  March 10, 2023
  
  - Core functionality implemented
  - Initial user testing
  - Feedback incorporated
  
  // And so on for remaining slides...
  
  
  // global/timelineStore.js (optional - for more complex state management)
  import { reactive } from 'vue'
  
  export const timelineState = reactive({
    projectEvents: [
      { date: '2023-01-15', title: 'Project Kickoff', slideIndex: 2 },
      { date: '2023-03-10', title: 'First Milestone', slideIndex: 5 },
      { date: '2023-05-20', title: 'Major Challenge', slideIndex: 8 },
      { date: '2023-07-15', title: 'Mid-Project Review', slideIndex: 12 },
      { date: '2023-10-05', title: 'Final Testing', slideIndex: 15 },
      { date: '2023-11-30', title: 'Project Delivery', slideIndex: 18 }
    ],
    
    // Methods to interact with timeline state
    getCurrentEvent(slideIndex) {
      return this.projectEvents.find(event => event.slideIndex === slideIndex) || this.projectEvents[0]
    },
    
    getEventPosition(event) {
      const startDate = new Date(this.projectEvents[0].date).getTime()
      const endDate = new Date(this.projectEvents[this.projectEvents.length - 1].date).getTime()
      const span = endDate - startDate
      const eventDate = new Date(event.date).getTime()
      
      return ((eventDate - startDate) / span) * 100
    }
  })