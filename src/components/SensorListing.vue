<script setup lang="ts">
import { ref, onMounted, computed } from 'vue'

import SensorEntry from './SensorEntry.vue'
import { type Measurement } from '../types'
const selected = ref(
  new Set(JSON.parse(localStorage.getItem('selected-sensors') ?? '[]') as string[])
)
const editMode = ref(false)
const loading = ref(true)
const error = ref(false)
const errorMessage = ref('')
const data = ref([] as Measurement[])

const updateSelected = () => {
  localStorage.setItem('selected-sensors', JSON.stringify([...selected.value]))
}

const toggleSelected = (mac: string) => {
  if (selected.value.has(mac)) {
    selected.value.delete(mac)
  } else {
    selected.value.add(mac)
  }
  updateSelected()
}

const fetchAndUpdate = () => {
  fetch('https://temperature.atk.works/api/v0/measurements')
    .then((response) => response.json())
    .then((json) => (data.value = json))
    .catch((err) => {
      error.value = true
      errorMessage.value = err.message
    })
    .finally(() => (loading.value = false))
}

onMounted(() => {
  fetchAndUpdate()
  setInterval(fetchAndUpdate, 3000)
})
</script>

<template>
  <div class="main">
    <button @click="editMode = !editMode">Toggle edit mode</button>
    <p v-if="loading">Loading...</p>
    <p v-else-if="error">Error: {{ errorMessage }}</p>
    <p v-else-if="!editMode && selected.size === 0">
      No sensors selected. Start by pressing "Toggle edit mode"
    </p>

    <template v-else v-for="entry in data" :key="entry.mac">
      <SensorEntry
        v-if="editMode || selected.has(entry.mac)"
        :data="entry"
        :editMode="editMode"
        :visible="selected.has(entry.mac)"
        @toggle-hide="toggleSelected"
      />
    </template>
  </div>
</template>

