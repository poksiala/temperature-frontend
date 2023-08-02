<script setup lang="ts">
import SensorListing from './components/SensorListing.vue'
</script>

<template>
  <main>
    <SensorListing />
  </main>
</template>

<style scoped>
@media (min-width: 1024px) {
  header {
    display: flex;
    place-items: center;
    padding-right: calc(var(--section-gap) / 2);
  }
}
</style>
