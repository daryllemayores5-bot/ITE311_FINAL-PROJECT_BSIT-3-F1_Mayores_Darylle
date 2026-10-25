<script>
export default {
  props: { event: { type: Object, required: true } },
  emits: ['edit', 'delete'],
  data() {
    return {
      showDetails: false,
      categoryColors: {
        Academic: '#365ce6', Sports: '#17815e', Cultural: '#a84ca0',
        Seminar: '#9165cf', Workshop: '#b16c17', 'School Activity': '#168595', Other: '#667085',
      },
    }
  },
  computed: {
    formattedDate() {
      return new Date(`${this.event.date}T00:00`).toLocaleDateString(undefined, {
        month: 'short', day: 'numeric', year: 'numeric',
      })
    },
    formattedTime() {
      const format = time => new Date(`${this.event.date}T${time}`).toLocaleTimeString(undefined, { hour: 'numeric', minute: '2-digit' })
      return `${format(this.event.startTime)} – ${format(this.event.endTime)}`
    },
  },
}
</script>

<template>
  <article class="event-card">
    <div class="event-card-top"><span class="category-label"><span class="category-dot" :style="{ backgroundColor: categoryColors[event.category] }"></span>{{ event.category }}</span><span :class="['status-badge', `status-${event.status.toLowerCase()}`]">{{ event.status }}</span></div>
    <h3>{{ event.title }}</h3>
    <dl class="event-facts">
      <div><dt><svg viewBox="0 0 24 24" aria-hidden="true"><rect x="4" y="5" width="16" height="16" rx="2"/><path d="M8 3v4m8-4v4M4 11h16"/></svg><span class="sr-only">Date</span></dt><dd>{{ formattedDate }}</dd></div>
      <div><dt><svg viewBox="0 0 24 24" aria-hidden="true"><circle cx="12" cy="12" r="9"/><path d="M12 7v5l3 2"/></svg><span class="sr-only">Time</span></dt><dd>{{ formattedTime }}</dd></div>
      <div><dt><svg viewBox="0 0 24 24" aria-hidden="true"><path d="M19 10c0 5-7 11-7 11S5 15 5 10a7 7 0 1 1 14 0Z"/><circle cx="12" cy="10" r="2"/></svg><span class="sr-only">Location</span></dt><dd>{{ event.location }}</dd></div>
      <div><dt><svg viewBox="0 0 24 24" aria-hidden="true"><circle cx="12" cy="8" r="4"/><path d="M4 21v-2a8 8 0 0 1 16 0v2"/></svg><span class="sr-only">Organizer</span></dt><dd>{{ event.organizer }}</dd></div>
    </dl>
    <!-- v-show keeps this element mounted, and changes its display property. -->
    <div v-show="showDetails" :id="`details-${event.id}`" class="event-description"><strong>About this event</strong><p>{{ event.description }}</p></div>
    <div class="event-actions"><button class="text-button details-button" :aria-expanded="showDetails" :aria-controls="`details-${event.id}`" @click="showDetails = !showDetails">{{ showDetails ? 'Hide Details' : 'View Details' }} <span aria-hidden="true">{{ showDetails ? '↑' : '↗' }}</span></button><button class="card-action" @click="$emit('edit', event)">Edit</button><button class="card-action delete-button" @click="$emit('delete', event)">Delete</button></div>
  </article>
</template>
