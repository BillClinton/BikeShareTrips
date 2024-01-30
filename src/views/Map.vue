<script setup lang="ts">
import { computed, onMounted, ref } from 'vue'
import { Loader } from '@googlemaps/js-api-loader'

const API_KEY = import.meta.env.VITE_APP_GOOGLE_MAPS_API_KEY

const phillyCityHall = { lat: 39.952695575063274, lng: -75.16359521464018 }

const loader = new Loader({ apiKey: API_KEY })

var myStyles = [
  {
    featureType: 'poi',
    elementType: 'labels',
    stylers: [{ visibility: 'off' }]
  }
]

const mapDiv = ref<HTMLElement | null>(null)
let map = ref(null)

function calculateBearing(startLatlng, endLatlng) {
  return endLatlng.lng() > startLatlng.lng()
}

onMounted(async () => {
  await loader.load()

  map.value = new google.maps.Map(mapDiv.value, {
    center: phillyCityHall,
    zoom: 14,
    disableDefaultUI: true,
    zoomControl: true,
    scaleControl: true,
    fullscreenControl: true,
    styles: myStyles
  })

  const trips = [
    {
      start: { lat: 39.944057, lng: -75.167591 },
      end: { lat: 39.9480993, lng: -75.1611755 }
    },
    {
      start: { lat: 39.94714153824316, lng: -75.16967676678941 },
      end: { lat: 39.94878110601268, lng: -75.16616885857079 }
    },
    {
      start: { lat: 39.946678679251924, lng: -75.16567880852082 },
      end: { lat: 39.95118717784482, lng: -75.16431089970993 }
    }
  ]

  // const directionsService = new google.maps.DirectionsService()
  const directionsRenderer = new google.maps.DirectionsRenderer()
  directionsRenderer.setMap(map.value)

  let lastBearing

  const bikeIconPathEast =
    'M312 32c-13.3 0-24 10.7-24 24s10.7 24 24 24h25.7l34.6 64H222.9l-27.4-38C191 99.7 183.7 96 176 96H120c-13.3 0-24 10.7-24 24s10.7 24 24 24h43.7l22.1 30.7-26.6 53.1c-10-2.5-20.5-3.8-31.2-3.8C57.3 224 0 281.3 0 352s57.3 128 128 128c65.3 0 119.1-48.9 127-112h49c8.5 0 16.3-4.5 20.7-11.8l84.8-143.5 21.7 40.1C402.4 276.3 384 312 384 352c0 70.7 57.3 128 128 128s128-57.3 128-128s-57.3-128-128-128c-13.5 0-26.5 2.1-38.7 6L375.4 48.8C369.8 38.4 359 32 347.2 32H312zM458.6 303.7l32.3 59.7c6.3 11.7 20.9 16 32.5 9.7s16-20.9 9.7-32.5l-32.3-59.7c3.6-.6 7.4-.9 11.2-.9c39.8 0 72 32.2 72 72s-32.2 72-72 72s-72-32.2-72-72c0-18.6 7-35.5 18.6-48.3zM133.2 368h65c-7.3 32.1-36 56-70.2 56c-39.8 0-72-32.2-72-72s32.2-72 72-72c1.7 0 3.4 .1 5.1 .2l-24.2 48.5c-9 18.1 4.1 39.4 24.3 39.4zm33.7-48l50.7-101.3 72.9 101.2-.1 .1H166.8zm90.6-128H365.9L317 274.8 257.4 192z'
  const bikeIconPathWest =
    'm 328,32 c 13.3,0 24,10.7 24,24 0,13.3 -10.7,24 -24,24 h -25.7 l -34.6,64 h 149.4 l 27.4,-38 C 449,99.7 456.3,96 464,96 h 56 c 13.3,0 24,10.7 24,24 0,13.3 -10.7,24 -24,24 h -43.7 l -22.1,30.7 26.6,53.1 c 10,-2.5 20.5,-3.8 31.2,-3.8 70.7,0 128,57.3 128,128 0,70.7 -57.3,128 -128,128 -65.3,0 -119.1,-48.9 -127,-112 h -49 c -8.5,0 -16.3,-4.5 -20.7,-11.8 L 230.5,212.7 208.8,252.8 C 237.6,276.3 256,312 256,352 256,422.7 198.7,480 128,480 57.3,480 0,422.7 0,352 0,281.3 57.3,224 128,224 c 13.5,0 26.5,2.1 38.7,6 L 264.6,48.8 C 270.2,38.4 281,32 292.8,32 Z m -146.6,271.7 -32.3,59.7 c -6.3,11.7 -20.9,16 -32.5,9.7 -11.6,-6.3 -16,-20.9 -9.7,-32.5 l 32.3,-59.7 c -3.6,-0.6 -7.4,-0.9 -11.2,-0.9 -39.8,0 -72,32.2 -72,72 0,39.8 32.2,72 72,72 39.8,0 72,-32.2 72,-72 0,-18.6 -7,-35.5 -18.6,-48.3 z M 506.8,368 h -65 c 7.3,32.1 36,56 70.2,56 39.8,0 72,-32.2 72,-72 0,-39.8 -32.2,-72 -72,-72 -1.7,0 -3.4,0.1 -5.1,0.2 l 24.2,48.5 c 9,18.1 -4.1,39.4 -24.3,39.4 z m -33.7,-48 -50.7,-101.3 -72.9,101.2 0.1,0.1 H 473.2 Z M 382.5,192 H 274.1 L 323,274.8 382.6,192 Z'

  function moveMarker(map, marker, latlng, nextLatlng) {
    marker.setPosition(latlng)

    if (nextLatlng) {
      const bearing = calculateBearing(
        new google.maps.LatLng(latlng),
        new google.maps.LatLng(nextLatlng)
      )

      if (bearing !== lastBearing) {
        let icon = marker.getIcon()
        icon.path = bearing ? bikeIconPathEast : bikeIconPathWest
        marker.setIcon(icon)
        lastBearing = bearing
      }
    }
  }

  let delay = 0

  function startTrip(index) {
    if (index >= trips.length) {
      return
    }

    let trip = trips[index]
    let directionsService = new google.maps.DirectionsService()
    let directionsRenderer = new google.maps.DirectionsRenderer({
      suppressMarkers: true,
      suppressPolylines: true
    })
    directionsRenderer.setMap(map.value)

    let request = {
      origin: trip.start,
      destination: trip.end,
      travelMode: 'BICYCLING'
    }

    directionsService.route(request, function (result, status) {
      if (status == 'OK') {
        let bikePath = result.routes[0].overview_path

        // Create a new polyline
        let polyline = new google.maps.Polyline({
          path: [],
          strokeColor: '#091F92',
          strokeOpacity: 1.0,
          strokeWeight: 4,
          map: map.value
        })

        directionsRenderer.setDirections(result)

        let marker = new google.maps.Marker({
          position: bikePath[0],
          map: map.value,
          icon: {
            fillColor: 'black',
            fillOpacity: 0.9,
            strokeWeight: 0,
            scale: 0.04,
            path: '',
            anchor: new google.maps.Point(200, 200)
          }
        })

        let i = 0
        let speed = 200 // Higher number = slower

        let interval = setInterval(function () {
          if (i < bikePath.length) {
            moveMarker(map.value, marker, bikePath[i], bikePath[i + 1])
            // Extend the polyline path by the current point
            let path = polyline.getPath()
            path.push(bikePath[i])

            i++
          } else {
            // Remove the marker from the map
            marker.setMap(null)
            // Remove the polyline from the map
            polyline.setMap(null)
            // Clear the directions from the directionsRenderer
            directionsRenderer.setDirections({ routes: [] })
            // Clear the interval
            clearInterval(interval)

            // Start the next trip
            startTrip(index + 1)
          }
        }, speed)
      } else {
        window.alert('Directions request failed due to ' + status)
      }
    })
  }

  // Start the first trip
  startTrip(0)
})
</script>

<template>
  <div class="map">
    <div ref="mapDiv" style="width: 800px; height: 600px; border: 1px solid yellow" />
  </div>
</template>

<style>
@media (min-width: 1024px) {
  .about {
    min-height: 100vh;
    display: flex;
    align-items: center;
  }
}
</style>
