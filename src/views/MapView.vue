<script setup lang="ts">
import { onMounted, ref } from 'vue'
import tripsData from '@/assets/trips2324.json'
import TripMap from '@/components/TripMap.vue'

interface Trip {
  start: { lat: number; lng: number }
  end: { lat: number; lng: number }
}

let trips = ref<Trip[] | null>(null)

onMounted(async () => {
  trips.value = tripsData.map((trip) => ({
    start: { lat: trip.checkOutLat, lng: trip.checkOutLon },
    end: { lat: trip.checkInLat, lng: trip.checkInLon }
  }))
})
</script>

<template>
  <Suspense>
    <!-- component with nested async dependencies -->
    <TripMap v-if="trips" :trips="trips" />

    <!-- loading state via #fallback slot -->
    <template #fallback> Loading... </template>
  </Suspense>
</template>
