<script>
import EventCard from './EventCard.vue'

export default {
  components: { EventCard },
  props: {
    statistics: { type: Object, required: true },
    upcomingEvents: { type: Array, required: true },
    todayLabel: { type: String, required: true },
  },
  emits: ['add', 'view-all', 'edit', 'delete'],
  data() {
    return {
      summaryCards: [
        { key: 'total', label: 'Total Events', caption: 'All your school events', icon: '▦', color: 'blue' },
        { key: 'upcoming', label: 'Upcoming Events', caption: 'Something to look forward to', icon: '↗', color: 'violet' },
        { key: 'ongoing', label: 'Ongoing Events', caption: 'Happening right now', icon: '◷', color: 'green' },
        { key: 'completed', label: 'Completed Events', caption: 'Moments made, memories kept', icon: '✓', color: 'gray' },
      ],
    }
  },
}
</script>

<template>
  <section class="welcome-banner" aria-labelledby="welcome-title">
    <div class="welcome-copy"><p class="banner-date">{{ todayLabel }}</p><h2 id="welcome-title">Great things happen<br />when we come together.</h2><p>From classroom ideas to campus celebrations,<br class="desktop-break" /> bring your next school event to life.</p><button class="button button-white" @click="$emit('add')">Create an event <span aria-hidden="true">↗</span></button></div>
    <div class="calendar-art" aria-hidden="true"><span class="art-orbit orbit-one"></span><span class="art-orbit orbit-two"></span><span class="art-spark spark-one">✦</span><span class="art-spark spark-two">＋</span><div class="paper-calendar"><div class="calendar-binding"><i></i><i></i></div><div class="calendar-paper-heading"><span>THE CAMPUS CALENDAR</span><strong>Make it happen.</strong></div><div class="calendar-squares"><i v-for="day in 21" :key="day" :class="{ 'marked-day': [5, 10, 17].includes(day) }">{{ day }}</i></div><div class="calendar-sticker">✓ All set for something great</div></div></div>
  </section>

  <div class="section-heading"><div><h2>At a glance</h2><p>Your events, automatically up to date.</p></div><span class="subtle-label">All time</span></div>
  <!-- Props flow from App to this component; interpolation displays each count. -->
  <section class="statistics-grid" aria-label="Event statistics">
    <article v-for="card in summaryCards" :key="card.key" :class="['stat-card', `stat-${card.color}`]"><div class="stat-top"><span>{{ card.label }}</span><span class="stat-icon" aria-hidden="true">{{ card.icon }}</span></div><strong class="stat-value">{{ statistics[card.key] }}</strong><p>{{ card.caption }}</p></article>
  </section>

  <section aria-labelledby="coming-up-title">
    <div class="section-heading"><div><h2 id="coming-up-title">Coming up on campus</h2><p>Ongoing events and the next dates to remember.</p></div><button class="text-button" @click="$emit('view-all')">View all events <span aria-hidden="true">→</span></button></div>
    <div v-if="upcomingEvents.length" class="event-grid"><EventCard v-for="event in upcomingEvents" :key="event.id" :event="event" @edit="$emit('edit', $event)" @delete="$emit('delete', $event)" /></div>
    <div v-else class="empty-state"><span class="empty-icon" aria-hidden="true">▦</span><h3>A fresh start for your next event</h3><p>No upcoming or ongoing events yet.<br />Add an event and give your campus something to look forward to.</p><button class="button button-primary" @click="$emit('add')"><span aria-hidden="true">＋</span> Create an event</button></div>
  </section>
</template>
