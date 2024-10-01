<template>
  <div :class="['container', currentBackgroundClass]">
    <div class="header">
      <h1>Weather app</h1>
      <button @click="fetchCurrentLocation" class="location-button">
        <img class="location-icon" src="/src/assets/icons/location.svg" alt="Location Icon" />
      </button>
      <div class="search-bar">
        <input
          type="text"
          v-model="city"
          placeholder="City..."
          class="search-input"
          @input="fetchCitySuggestions"
        />
        <button @click="searchByCity" class="search-button">Search</button>
      </div>
    </div>

    <div class="main-section"></div>
    <div class="section"></div>
    <div class="section"></div>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'
import axios from 'axios'

const apikey = 'b09588d826e97199e406a30e962cdb60'

const weatherData = ref(null)
const suggestions = ref([])
const city = ref('')

const backgroundMapping = {
  '01d': 'sunny-day',
  '01n': 'sunny-night',
  '02d': 'few-clouds-day',
  '02n': 'few-clouds-night',
  '03d': 'scattered-clouds-day',
  '03n': 'scattered-clouds-night',
  '04d': 'broken-clouds-day',
  '04n': 'broken-clouds-night',
  '09d': 'shower-rain-day',
  '09n': 'shower-rain-night',
  '10d': 'rain-day',
  '10n': 'rain-night',
  '11d': 'thunderstorm-day',
  '11n': 'thunderstorm-night',
  '13d': 'snow-day',
  '13n': 'snow-night',
  '50d': 'mist-day',
  '50n': 'mist-night'
}

const currentBackgroundClass = computed(() =>
  weatherData.value ? backgroundMapping[weatherData.value.weather[0].icon] : ''
)

const fetchCitySuggestions = async () => {
  if (!city.value || city.value.length < 3) {
    suggestions.value = []
    return
  }
  try {
    const url = `http://api.openweathermap.org/data/2.5/find?q=${city.value}&appid=${apikey}`
    const response = await axios.get(url)
    const uniqueCities = new Set()
    suggestions.value = response.data.list
      .map((city) => ({
        name: city.name,
        country: city.sys.country
      }))
      .filter((city) => {
        const key = `${city.name}, ${city.country}`
        if (!uniqueCities.has(key)) {
          uniqueCities.add(key)
          return true
        }
        return false
      })
  } catch (error) {
    suggestions.value = []
  }
}

const selectCity = (selectedCity) => {
  city.value = `${selectedCity.name}, ${selectedCity.country}`
  searchByCity()
  suggestions.value = []
}

const fetchCurrentLocationWeather = async () => {
  if (!navigator.geolocation) {
    console.error('Geolocation is not supported by this browser.')
    return
  }
  navigator.geolocation.getCurrentPosition(
    async (position) => {
      const { latitude, longitude } = position.coords
      const url = `http://api.openweathermap.org/data/2.5/weather?lat=${latitude}&lon=${longitude}&appid=${apikey}`
      try {
        await fetchWeatherData(url)
        hoverBackgroundClass.value = backgroundMapping[weatherData.value.weather[0].icon] || ''
      } catch (error) {
        console.error('Error fetching weather data for current location:', error)
      }
    },
    (error) => {
      console.error('Error getting geolocation:', error)
    }
  )
}

const fetchWeatherData = async (url) => {
  try {
    const response = await axios.get(url)
    weatherData.value = response.data
    hoverBackgroundClass.value = backgroundMapping[weatherData.value.weather[0].icon] || ''
    await fetchForecast(weatherData.value.name)
  } catch (error) {
    console.error('Error fetching weather data:', error)
  }
}

const fetchForecast = async (city) => {
  const urlcast = `http://api.openweathermap.org/data/2.5/forecast?q=${city}&appid=${apikey}`
  try {
    const response = await axios.get(urlcast)
    const forecast = response.data
    hourForecast(forecast)
    dayForecast(forecast)
  } catch (error) {
    console.error('Error fetching forecast data:', error)
  }
}

const searchByCity = async () => {
  if (!city.value) return
  try {
    const urlsearch = `http://api.openweathermap.org/data/2.5/weather?q=${city.value}&appid=${apikey}`
    const response = await axios.get(urlsearch)
    weatherData.value = response.data
    await fetchForecast(weatherData.value.name)
    hoverBackgroundClass.value = backgroundMapping[weatherData.value.weather[0].icon] || ''
  } catch (error) {
    console.error('Error fetching city suggestions:', error)
  }
}
</script>

<style scoped></style>
