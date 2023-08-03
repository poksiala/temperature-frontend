<script setup lang="ts">
import { computed, ref } from 'vue'
import { type Measurement } from '../types'

const emit = defineEmits<{
  'toggle-hide': [mac: string]
}>()

const props = defineProps<{
  editMode?: boolean
  visible?: boolean
  data: Measurement
}>()

const renameMode = ref(false)
const alias = ref(localStorage.getItem('alias-' + props.data.mac))
const mac = computed(() =>
  props.data.mac
    .toUpperCase()
    .replace(/(.{2})/g, '$1:')
    .slice(0, -1)
)
const title = computed(() => alias.value ?? mac.value)

const temperature = computed(() => String(props.data.temperature / 100) + ' °C')

const humidity = computed(() => String(props.data.humidity / 100) + ' %')

const timeSince = computed(() => {
  const seconds = Math.floor((Date.now() - props.data.ts * 1000) / 1000)
  const minutes = Math.floor(seconds / 60)
  const hours = Math.floor(minutes / 60)
  const days = Math.floor(hours / 24)
  const weeks = Math.floor(days / 7)
  const months = Math.floor(days / 30)
  const years = Math.floor(days / 365)

  if (years > 0) {
    return years + ' years ago'
  } else if (months > 0) {
    return months + ' months ago'
  } else if (weeks > 0) {
    return weeks + ' weeks ago'
  } else if (days > 0) {
    return days + ' days ago'
  } else if (hours > 0) {
    return hours + ' hours ago'
  } else if (minutes > 0) {
    return minutes + ' minutes ago'
  } else {
    return seconds + ' seconds ago'
  }
})

const submitName = () => {
  if (alias.value === '') {
    alias.value = null
  }
  if (alias.value === null) {
    localStorage.removeItem('alias-' + props.data.mac)
  } else {
    localStorage.setItem('alias-' + props.data.mac, alias.value)
  }
  renameMode.value = false
}
</script>

<template>
  <div :class="props.visible ? '' : 'hidden'">
    <h2 :class="props.visible ? '' : 'hidden'" >{{ title }}</h2>
    <input
      v-if="editMode && renameMode"
      v-model="alias"
      @keydown.enter="submitName"
      @keydown.esc="submitName"
    />
    <button v-if="editMode && renameMode" @click="submitName">Submit</button>
    <div v-if="editMode && !renameMode">
      <button @click="renameMode = true">Rename</button>
      <button @click="$emit('toggle-hide', props.data.mac)">
        {{ props.visible ? 'Hide' : 'Show' }}
      </button>
    </div>
    <ul>
      <li>Temperature: {{ temperature }}</li>
      <li>Humidity: {{ humidity }}</li>
      <li>{{ timeSince }}</li>
    </ul>
  </div>
</template>

<style scoped>


.hidden {
  color: gray;
}
</style>
