<script>
export default {
  props: {
    event: { type: Object, default: null },
    categories: { type: Array, required: true },
  },
  emits: ['save', 'cancel'],
  data() {
    return {
      // Copy editable fields, so typing never mutates a parent's prop.
      form: {
        title: this.event?.title || '',
        description: this.event?.description || '',
        category: this.event?.category || '',
        date: this.event?.date || '',
        startTime: this.event?.startTime || '',
        endTime: this.event?.endTime || '',
        location: this.event?.location || '',
        organizer: this.event?.organizer || '',
      },
      errors: {},
    }
  },
  mounted() {
    // A template ref is a DOM reference, different from Composition API ref().
    this.$refs.titleInput.focus()
  },
  methods: {
    validateForm() {
      const errors = {}
      const labels = {
        title: 'Event title', description: 'Description', category: 'Category',
        date: 'Event date', startTime: 'Start time', endTime: 'End time',
        location: 'Location', organizer: 'Organizer',
      }
      for (const field in labels) {
        if (!this.form[field].trim()) errors[field] = `${labels[field]} is required.`
      }
      if (this.form.startTime && this.form.endTime && this.form.endTime <= this.form.startTime) {
        errors.endTime = 'End time must be later than start time.'
      }
      this.errors = errors
      return Object.keys(errors).length === 0
    },
    submitForm() {
      if (!this.validateForm()) {
        this.$nextTick(() => this.$el.querySelector('[aria-invalid="true"]')?.focus())
        return
      }
      const event = { ...this.form }
      for (const field in event) event[field] = event[field].trim()
      this.$emit('save', event)
    },
  },
}
</script>

<template>
  <section class="form-panel" aria-labelledby="form-title">
    <div class="form-heading"><div><h2 id="form-title">{{ event ? 'Edit event' : 'Create a new event' }}</h2><p>Fill in the details. All fields are required.</p></div><span class="form-tag">{{ event ? 'EDITING' : 'NEW EVENT' }}</span></div>
    <!-- novalidate lets our simple Vue validation show consistent messages. -->
    <form novalidate @submit.prevent="submitForm">
      <div class="form-grid">
        <div class="field full-width"><label for="event-title">Event Title</label><input id="event-title" ref="titleInput" v-model="form.title" placeholder="e.g. School Foundation Day" required :aria-invalid="!!errors.title" :aria-describedby="errors.title ? 'title-error' : undefined" /><small v-if="errors.title" id="title-error" class="field-error">{{ errors.title }}</small></div>
        <div class="field full-width"><label for="event-description">Event Description</label><textarea id="event-description" v-model="form.description" rows="3" placeholder="What is this event all about?" required :aria-invalid="!!errors.description" :aria-describedby="errors.description ? 'description-error' : undefined"></textarea><small v-if="errors.description" id="description-error" class="field-error">{{ errors.description }}</small></div>
        <div class="field"><label for="event-category">Event Category</label><select id="event-category" v-model="form.category" required :aria-invalid="!!errors.category" :aria-describedby="errors.category ? 'category-error' : undefined"><option disabled value="">Select a category</option><option v-for="category in categories" :key="category" :value="category">{{ category }}</option></select><small v-if="errors.category" id="category-error" class="field-error">{{ errors.category }}</small></div>
        <div class="field"><label for="event-date">Event Date</label><input id="event-date" v-model="form.date" type="date" required :aria-invalid="!!errors.date" :aria-describedby="errors.date ? 'date-error' : undefined" /><small v-if="errors.date" id="date-error" class="field-error">{{ errors.date }}</small></div>
        <div class="field"><label for="event-start">Start Time</label><input id="event-start" v-model="form.startTime" type="time" required :aria-invalid="!!errors.startTime" :aria-describedby="errors.startTime ? 'start-error' : undefined" /><small v-if="errors.startTime" id="start-error" class="field-error">{{ errors.startTime }}</small></div>
        <div class="field"><label for="event-end">End Time</label><input id="event-end" v-model="form.endTime" type="time" required :aria-invalid="!!errors.endTime" :aria-describedby="errors.endTime ? 'end-error' : undefined" /><small v-if="errors.endTime" id="end-error" class="field-error">{{ errors.endTime }}</small></div>
        <div class="field"><label for="event-location">Event Location</label><input id="event-location" v-model="form.location" placeholder="e.g. School Gymnasium" required :aria-invalid="!!errors.location" :aria-describedby="errors.location ? 'location-error' : undefined" /><small v-if="errors.location" id="location-error" class="field-error">{{ errors.location }}</small></div>
        <div class="field"><label for="event-organizer">Event Organizer</label><input id="event-organizer" v-model="form.organizer" placeholder="e.g. Student Council" required :aria-invalid="!!errors.organizer" :aria-describedby="errors.organizer ? 'organizer-error' : undefined" /><small v-if="errors.organizer" id="organizer-error" class="field-error">{{ errors.organizer }}</small></div>
      </div>
      <p v-if="Object.keys(errors).length" class="field-error validation-summary" role="alert">Please correct the highlighted fields before saving.</p>
      <div class="form-actions"><span>Times use your local timezone.</span><button type="button" class="button button-secondary" @click="$emit('cancel')">Cancel</button><button type="submit" class="button button-primary">{{ event ? 'Save Changes' : 'Save Event' }}</button></div>
    </form>
  </section>
</template>
