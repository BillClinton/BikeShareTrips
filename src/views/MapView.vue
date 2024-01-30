<script setup lang="ts">
import Map from './Map.vue'
</script>

<template>
  <Suspense>
    <!-- component with nested async dependencies -->
    <Map />

    <!-- loading state via #fallback slot -->
    <template #fallback> Loading... </template>
  </Suspense>
</template>
