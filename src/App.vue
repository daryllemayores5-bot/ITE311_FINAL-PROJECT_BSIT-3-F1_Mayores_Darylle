<script>
import Dashboard from './components/Dashboard.vue'
import EventForm from './components/EventForm.vue'
import EventList from './components/EventList.vue'

const STORAGE_KEY = 'school-event-management-events'
const CATEGORIES = ['Academic', 'Sports', 'Cultural', 'Seminar', 'Workshop', 'School Activity', 'Other']

export default {
  components: { Dashboard, EventForm, EventList },
  data() {
    // Options API data becomes reactive: Vue updates the screen when it changes.
    return {
      events: [],
      categories: CATEGORIES,
      activePage: 'Dashboard',
      showForm: false,
      editingEvent: null,
      currentTime: Date.now(),
      clockTimer: null,
      notice: '',
      storageWarning: '',
    }
  },
  computed: {
    // Count the reactive events array, including events restored from localStorage.
    totalEvents() {
      return this.events.length
    },
    categoryCounts() {
      // Include every existing category, even when no events match it.
      return this.categories.map(category => ({
        category,
        count: this.events.filter(event => event.category === category).length,
      }))
    },
    // Status is calculated for display only; it is never stored with an event.
    eventsWithStatus() {
      return this.events.map(event => ({ ...event, status: this.getEventStatus(event) }))
    },
    statistics() {
      return {
        total: this.totalEvents,
        upcoming: this.eventsWithStatus.filter(event => event.status === 'Upcoming').length,
        ongoing: this.eventsWithStatus.filter(event => event.status === 'Ongoing').length,
        completed: this.eventsWithStatus.filter(event => event.status === 'Completed').length,
      }
    },
    upcomingEvents() {
      return this.eventsWithStatus
        .filter(event => event.status !== 'Completed')
        .sort((a, b) => this.eventStart(a) - this.eventStart(b))
        .slice(0, 3)
    },
    todayLabel() {
      return new Date(this.currentTime).toLocaleDateString(undefined, {
        weekday: 'long', month: 'long', day: 'numeric', year: 'numeric',
      })
    },
  },
  watch: {
    // A deep watcher also detects edits inside event objects.
    events: {
      deep: true,
      handler(events) {
        try {
          localStorage.setItem(STORAGE_KEY, JSON.stringify(events))
          this.storageWarning = ''
        } catch {
          this.storageWarning = 'Your browser could not save these changes. They will last only until this page is refreshed.'
        }
      },
    },
  },
  created() {
    this.loadEvents()
  },
  mounted() {
    this.clockTimer = window.setInterval(() => {
      this.currentTime = Date.now()
    }, 60000)
    window.addEventListener('focus', this.refreshClock)
  },
  beforeUnmount() {
    window.clearInterval(this.clockTimer)
    window.removeEventListener('focus', this.refreshClock)
  },
  methods: {
    refreshClock() {
      this.currentTime = Date.now()
    },
    eventStart(event) {
      // No UTC suffix: the selected date and time belong to the user's local timezone.
      return new Date(`${event.date}T${event.startTime}`).getTime()
    },
    getEventStatus(event) {
      const start = this.eventStart(event)
      const end = new Date(`${event.date}T${event.endTime}`).getTime()
      if (this.currentTime < start) return 'Upcoming'
      if (this.currentTime <= end) return 'Ongoing'
      return 'Completed'
    },
    isValidStoredEvent(event) {
      if (!event || typeof event !== 'object') return false
      const textFields = ['title', 'description', 'category', 'date', 'startTime', 'endTime', 'location', 'organizer']
      if (!textFields.every(field => typeof event[field] === 'string' && event[field].trim())) return false
      if (!CATEGORIES.includes(event.category)) return false
      if (!((typeof event.id === 'string' && event.id.trim()) || (typeof event.id === 'number' && Number.isFinite(event.id)))) return false
      if (!/^\d{4}-\d{2}-\d{2}$/.test(event.date)) return false
      if (!/^([01]\d|2[0-3]):[0-5]\d$/.test(event.startTime) || !/^([01]\d|2[0-3]):[0-5]\d$/.test(event.endTime)) return false
      const date = new Date(`${event.date}T00:00`)
      const [year, month, day] = event.date.split('-').map(Number)
      return date.getFullYear() === year && date.getMonth() + 1 === month && date.getDate() === day && event.endTime > event.startTime
    },
    loadEvents() {
      try {
        const stored = localStorage.getItem(STORAGE_KEY)
        if (!stored) return
        const parsed = JSON.parse(stored)
        if (!Array.isArray(parsed)) throw new Error('Invalid event collection')
        const ids = new Set()
        const validEvents = parsed.filter(event => {
          if (!this.isValidStoredEvent(event) || ids.has(String(event.id))) return false
          ids.add(String(event.id))
          return true
        })
        // Select known fields, so imported/stale calculated statuses are discarded.
        this.events = validEvents.map(event => ({
          id: event.id, title: event.title, description: event.description,
          category: event.category, date: event.date, startTime: event.startTime,
          endTime: event.endTime, location: event.location, organizer: event.organizer,
        }))
        if (validEvents.length !== parsed.length) {
          this.notice = 'Some saved records were invalid and could not be loaded. Valid events are still available.'
        }
      } catch {
        this.storageWarning = 'Saved events could not be loaded. You can still create events; saving a new event will replace the unreadable data.'
      }
    },
    navigate(page) {
      this.activePage = page
      this.cancelEdit()
      this.notice = ''
    },
    openAddForm() {
      this.activePage = 'Events'
      this.editingEvent = null
      this.showForm = true
      this.notice = ''
    },
    saveEvent(event) {
      if (this.editingEvent) this.updateEvent(event)
      else this.addEvent(event)
      this.refreshClock()
      this.cancelEdit()
    },
    addEvent(event) {
      this.events.push({ ...event, id: crypto.randomUUID() })
      this.notice = 'Event added successfully.'
    },
    updateEvent(event) {
      const index = this.events.findIndex(item => item.id === this.editingEvent.id)
      if (index !== -1) this.events.splice(index, 1, { ...event, id: this.editingEvent.id })
      this.notice = 'Event updated successfully.'
    },
    editEvent(event) {
      this.activePage = 'Events'
      this.editingEvent = event
      this.showForm = true
      this.notice = ''
      window.scrollTo({ top: 0 })
    },
    deleteEvent(event) {
      if (!window.confirm(`Delete “${event.title}”? This cannot be undone.`)) return
      this.events = this.events.filter(item => item.id !== event.id)
      if (this.editingEvent?.id === event.id) this.cancelEdit()
      this.notice = 'Event deleted successfully.'
    },
    cancelEdit() {
      this.showForm = false
      this.editingEvent = null
    },
  },
}
</script>

<template>
  <div class="app-layout">
    <a class="skip-link" href="#main-content">Skip to content</a>
    <aside class="sidebar">
      <a class="brand" href="#" @click.prevent="navigate('Dashboard')" aria-label="School Event Management home">
        <span class="brand-icon" aria-hidden="true">
          <svg viewBox="0 0 24 24"><rect x="4" y="5" width="16" height="16" rx="3"/><path d="M8 3v4m8-4v4M4 11h16m-11 5h6"/></svg>
        </span>
        <span>School Event<span class="brand-subtitle">Management System</span></span>
      </a>
      <p class="nav-label">WORKSPACE</p>
      <nav aria-label="Main navigation">
        <button :class="['nav-item', { active: activePage === 'Dashboard' }]" :aria-current="activePage === 'Dashboard' ? 'page' : undefined" @click="navigate('Dashboard')">
          <svg viewBox="0 0 24 24" aria-hidden="true"><rect x="3" y="3" width="7" height="7" rx="1.5"/><rect x="14" y="3" width="7" height="7" rx="1.5"/><rect x="3" y="14" width="7" height="7" rx="1.5"/><rect x="14" y="14" width="7" height="7" rx="1.5"/></svg>
          Dashboard
        </button>
        <button :class="['nav-item', { active: activePage === 'Events' }]" :aria-current="activePage === 'Events' ? 'page' : undefined" @click="navigate('Events')">
          <svg viewBox="0 0 24 24" aria-hidden="true"><rect x="3" y="5" width="18" height="16" rx="2"/><path d="M7 3v4m10-4v4M3 11h18m-13 5h2m4 0h2"/></svg>
          Events <span class="nav-count">{{ events.length }}</span>
        </button>
      </nav>
      <div class="sidebar-note">
        <span class="small-mark" aria-hidden="true">✦</span>
        <strong>Good events start here.</strong>
        <p>A little planning.<br />A lot of campus memories.</p>
      </div>
      <div class="sidebar-footer"><span class="course-avatar">SE</span><span>School Event Management<small>ITE 311 · Vue.js Project</small></span></div>
    </aside>

    <div class="main-shell">
      <header class="topbar"><span>Workspace <span class="breadcrumb-divider">/</span> <strong>{{ activePage }}</strong></span><span class="local-indicator"><span></span> Local workspace</span></header>
      <main id="main-content" class="main-content" tabindex="-1">
        <div class="page-heading">
          <div><p class="eyebrow">YOUR CAMPUS, CONNECTED</p><h1>{{ activePage === 'Dashboard' ? 'Dashboard' : 'School events' }}</h1><p class="page-subtitle">{{ activePage === 'Dashboard' ? 'A little overview of everything happening at school.' : 'Plan, organize, and keep every school event on track.' }}</p></div>
          <button class="button button-primary" @click="openAddForm"><span aria-hidden="true">＋</span> Add Event</button>
        </div>

        <p v-if="storageWarning" class="message message-warning" role="alert">{{ storageWarning }}</p>
        <p v-if="notice" class="message message-success" role="status">{{ notice }}</p>

        <!-- v-if/v-else switches pages without Vue Router. -->
        <Dashboard v-if="activePage === 'Dashboard'" :statistics="statistics" :category-counts="categoryCounts" :upcoming-events="upcomingEvents" :today-label="todayLabel" @add="openAddForm" @view-all="navigate('Events')" @edit="editEvent" @delete="deleteEvent" />
        <template v-else>
          <EventForm v-if="showForm" :key="editingEvent ? editingEvent.id : 'new'" :event="editingEvent" :categories="categories" @save="saveEvent" @cancel="cancelEdit" />
          <EventList :events="eventsWithStatus" :categories="categories" @add="openAddForm" @edit="editEvent" @delete="deleteEvent" />
        </template>
        <footer class="page-footer"><span>Made for a more organized campus.</span><span>School Event Management System</span></footer>
      </main>
    </div>
  </div>
</template>
