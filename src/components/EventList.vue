<script>
import EventCard from './EventCard.vue'

export default {
  components: { EventCard },
  props: {
    events: { type: Array, required: true },
    categories: { type: Array, required: true },
  },
  emits: ['add', 'edit', 'delete'],
  data() {
    return { searchQuery: '', selectedCategory: 'All', selectedStatus: 'All', selectedSort: 'date-earliest' }
  },
  computed: {
    // Computed properties refresh when their reactive dependencies change.
    filteredEvents() {
      return this.events.filter(event => {
        const matchesTitle = event.title.toLowerCase().includes(this.searchQuery.trim().toLowerCase())
        const matchesCategory = this.selectedCategory === 'All' || event.category === this.selectedCategory
        const matchesStatus = this.selectedStatus === 'All' || event.status === this.selectedStatus
        return matchesTitle && matchesCategory && matchesStatus
      })
    },
    sortedEvents() {
      // Sort a copy so the original events array keeps its order.
      const eventsToSort = [...this.filteredEvents]
      return eventsToSort.sort((a, b) => {
        // Ignore capitalization when comparing event titles.
        if (this.selectedSort === 'title-asc') return a.title.localeCompare(b.title, undefined, { sensitivity: 'base' })
        if (this.selectedSort === 'title-desc') return b.title.localeCompare(a.title, undefined, { sensitivity: 'base' })
        const firstDate = new Date(`${a.date}T${a.startTime}`)
        const secondDate = new Date(`${b.date}T${b.startTime}`)
        return this.selectedSort === 'date-latest' ? secondDate - firstDate : firstDate - secondDate
      })
    },
    sortLabel() {
      const labels = {
        'date-earliest': 'Soonest first',
        'date-latest': 'Latest first',
        'title-asc': 'Title: A to Z',
        'title-desc': 'Title: Z to A',
      }
      return labels[this.selectedSort]
    },
    hasFilters() {
      return this.searchQuery !== '' || this.selectedCategory !== 'All' || this.selectedStatus !== 'All'
    },
  },
  methods: {
    resetFilters() {
      this.searchQuery = ''
      this.selectedCategory = 'All'
      this.selectedStatus = 'All'
    },
  },
}
</script>

<template>
  <section aria-label="Event collection">
    <div class="filter-bar">
      <div class="field search-field"><label for="search-events">Search events</label><div class="search-input"><svg viewBox="0 0 24 24" aria-hidden="true"><circle cx="10.5" cy="10.5" r="6.5"/><path d="m16 16 4 4"/></svg><input id="search-events" v-model="searchQuery" type="search" placeholder="Search by event title…" /></div></div>
      <div class="field"><label for="category-filter">Category</label><select id="category-filter" v-model="selectedCategory"><option value="All">All categories</option><option v-for="category in categories" :key="category" :value="category">{{ category }}</option></select></div>
      <div class="field"><label for="status-filter">Status</label><select id="status-filter" v-model="selectedStatus"><option value="All">All statuses</option><option>Upcoming</option><option>Ongoing</option><option>Completed</option></select></div>
      <div class="field"><label for="event-sort">Sort by</label><select id="event-sort" v-model="selectedSort"><option value="date-earliest">Date: Earliest First</option><option value="date-latest">Date: Latest First</option><option value="title-asc">Title: A to Z</option><option value="title-desc">Title: Z to A</option></select></div>
    </div>
    <div class="collection-heading"><h2>All events <span class="count-pill">{{ filteredEvents.length }}</span></h2><button v-if="hasFilters" class="text-button" @click="resetFilters">Clear filters</button><span v-else class="subtle-label">{{ sortLabel }}</span></div>
    <!-- Stable keys help Vue track cards correctly after an edit or deletion. -->
    <div v-if="sortedEvents.length" class="event-grid"><EventCard v-for="event in sortedEvents" :key="event.id" :event="event" @edit="$emit('edit', $event)" @delete="$emit('delete', $event)" /></div>
    <div v-else class="empty-state"><span class="empty-icon" aria-hidden="true">▦</span><h3>{{ events.length ? 'No matching events' : 'Your next great event starts here' }}</h3><p>{{ events.length ? 'Try a different title, category, or status.' : 'Keep every date, detail, and campus celebration in one place.' }}</p><button v-if="events.length" class="button button-secondary" @click="resetFilters">Clear filters</button><button v-else class="button button-primary" @click="$emit('add')"><span aria-hidden="true">＋</span> Add your first event</button></div>
  </section>
</template>
