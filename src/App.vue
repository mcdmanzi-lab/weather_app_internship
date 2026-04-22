<script setup>
import { computed, onMounted, ref } from 'vue'

const API_KEY = import.meta.env.VITE_WEATHERAPI_KEY
const WEATHER_BASE_URL = 'https://api.weatherapi.com/v1/current.json'
const LAST_CITY_KEY = 'weather_last_city'

const city = ref('')
const weather = ref(null)
const loading = ref(false)
const errorMessage = ref('')
const sourceLabel = ref('')

const hasApiKey = computed(() => Boolean(API_KEY))
const canSearch = computed(() => city.value.trim().length > 0 && hasApiKey.value)

async function loadWeatherByCity(targetCity) {
  const trimmedCity = targetCity.trim()
  if (!trimmedCity) return

  loading.value = true
  errorMessage.value = ''

  try {
    const params = new URLSearchParams({
      key: API_KEY,
      q: trimmedCity,
      aqi: 'no'
    })
    const response = await fetch(`${WEATHER_BASE_URL}?${params.toString()}`)
    const data = await response.json()

    if (!response.ok) {
      throw new Error(data?.message || 'Failed to fetch weather data.')
    }

    weather.value = data
    sourceLabel.value = 'Search result'
    city.value = data.location?.name || trimmedCity
    localStorage.setItem(LAST_CITY_KEY, city.value)
  } catch (error) {
    weather.value = null
    errorMessage.value = `Could not find weather for "${trimmedCity}".`
    if (error instanceof Error && error.message) {
      errorMessage.value = error.message
    }
  } finally {
    loading.value = false
  }
}

async function loadWeatherByCoords(latitude, longitude) {
  loading.value = true
  errorMessage.value = ''

  try {
    const params = new URLSearchParams({
      key: API_KEY,
      q: `${latitude},${longitude}`,
      aqi: 'no'
    })
    const response = await fetch(`${WEATHER_BASE_URL}?${params.toString()}`)
    const data = await response.json()

    if (!response.ok) {
      throw new Error(data?.message || 'Failed to fetch weather data.')
    }

    weather.value = data
    sourceLabel.value = 'Current location'
    city.value = data.location?.name || city.value
    localStorage.setItem(LAST_CITY_KEY, city.value)
  } catch (error) {
    weather.value = null
    errorMessage.value = 'Unable to load weather for your current location.'
    if (error instanceof Error && error.message) {
      errorMessage.value = error.message
    }
  } finally {
    loading.value = false
  }
}

function handleSearch() {
  if (!canSearch.value || loading.value) return
  loadWeatherByCity(city.value)
}

function useCurrentLocation() {
  if (!hasApiKey.value || loading.value) return
  if (!navigator.geolocation) {
    errorMessage.value = 'Geolocation is not supported by this browser.'
    return
  }

  errorMessage.value = ''
  navigator.geolocation.getCurrentPosition(
    (position) => {
      loadWeatherByCoords(position.coords.latitude, position.coords.longitude)
    },
    (err) => {
      errorMessage.value = err.message || 'Unable to access your location.'
    },
    { enableHighAccuracy: true, timeout: 10000 }
  )
}

onMounted(() => {
  if (!hasApiKey.value) return

  const lastCity = localStorage.getItem(LAST_CITY_KEY)
  if (lastCity) {
    city.value = lastCity
    loadWeatherByCity(lastCity)
  }
})
</script>

<template>
  <main class="app">
    <section class="card">
      <h1>Weather App</h1>
      <p class="subtitle">Search any city and get live weather data.</p>

      <p v-if="!hasApiKey" class="error">
        Missing API key. Create a <code>.env</code> file with
        <code>VITE_WEATHERAPI_KEY=your_key</code>.
      </p>

      <form class="search-row" @submit.prevent="handleSearch">
        <input
          v-model="city"
          type="text"
          placeholder="Enter city name (e.g. Nairobi)"
          :disabled="loading || !hasApiKey"
        />
        <button type="submit" :disabled="!canSearch || loading">
          {{ loading ? 'Loading...' : 'Search' }}
        </button>
      </form>

      <button class="location-btn" :disabled="loading || !hasApiKey" @click="useCurrentLocation">
        Use Current Location
      </button>

      <p v-if="errorMessage" class="error">{{ errorMessage }}</p>

      <article v-if="weather" class="weather-panel">
        <div class="weather-header">
          <h2>{{ weather.location?.name }}, {{ weather.location?.country }}</h2>
          <span class="source">{{ sourceLabel }}</span>
        </div>

        <div class="summary">
          <img
            v-if="weather.current?.condition?.icon"
            :src="`https:${weather.current.condition.icon}`"
            :alt="weather.current?.condition?.text || 'Weather icon'"
            width="96"
            height="96"
          />
          <div>
            <p class="temp">{{ Math.round(weather.current?.temp_c) }}°C</p>
            <p class="condition">{{ weather.current?.condition?.text }}</p>
          </div>
        </div>

        <div class="metrics">
          <p><strong>Feels like:</strong> {{ Math.round(weather.current?.feelslike_c) }}°C</p>
          <p><strong>Humidity:</strong> {{ weather.current?.humidity }}%</p>
          <p><strong>Wind speed:</strong> {{ weather.current?.wind_kph }} km/h</p>
          <p><strong>Pressure:</strong> {{ weather.current?.pressure_mb }} hPa</p>
        </div>
      </article>
    </section>
  </main>
</template>

<style scoped>
.app {
  min-height: 100vh;
  display: grid;
  place-items: center;
  padding: 1rem;
}

.card {
  width: min(680px, 100%);
  background: #ffffff;
  border-radius: 16px;
  padding: 1.25rem;
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.08);
}

h1 {
  margin: 0;
  font-size: 1.8rem;
}

.subtitle {
  margin: 0.4rem 0 1rem;
  color: #5f6b7a;
}

.search-row {
  display: flex;
  gap: 0.6rem;
  flex-wrap: wrap;
}

input {
  flex: 1;
  min-width: 220px;
  border: 1px solid #d0d7e2;
  border-radius: 10px;
  padding: 0.7rem 0.85rem;
  font-size: 1rem;
}

button {
  border: none;
  border-radius: 10px;
  background: #1f6feb;
  color: #fff;
  padding: 0.72rem 1rem;
  cursor: pointer;
  font-weight: 600;
}

button:disabled {
  cursor: not-allowed;
  opacity: 0.6;
}

.location-btn {
  margin-top: 0.75rem;
  background: #0a7f5a;
}

.error {
  margin-top: 0.9rem;
  color: #b42318;
}

.weather-panel {
  margin-top: 1rem;
  border: 1px solid #e3e7ee;
  border-radius: 12px;
  padding: 1rem;
  background: #f8fbff;
}

.weather-header {
  display: flex;
  align-items: baseline;
  justify-content: space-between;
  gap: 0.8rem;
}

.weather-header h2 {
  margin: 0;
}

.source {
  color: #4a5a70;
  font-size: 0.92rem;
}

.summary {
  display: flex;
  align-items: center;
  gap: 0.7rem;
}

.temp {
  margin: 0;
  font-size: 2rem;
  font-weight: 700;
}

.condition {
  margin: 0;
  text-transform: capitalize;
  color: #3f4a59;
}

.metrics {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(150px, 1fr));
  gap: 0.45rem;
  margin-top: 0.8rem;
}

.metrics p {
  margin: 0;
}

@media (max-width: 540px) {
  .card {
    padding: 1rem;
  }

  .summary {
    align-items: flex-start;
  }
}
</style>
