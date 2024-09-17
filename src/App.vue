<template>
  <div :class="['container', currentBackgroundClass, hoverBackgroundClass]">
    <div class="header">
      <h1>WEATHER APP</h1>
      <button @click="fetchCurrentLocationWeather" class="location-button">
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
        <ul v-if="suggestions.length" class="suggestions-list">
          <li
            v-for="(suggestion, index) in suggestions"
            :key="index"
            @click="selectCity(suggestion)"
            class="suggestion-item"
          >
            {{ suggestion.name }}, {{ suggestion.country }}
          </li>
        </ul>
      </div>
    </div>

    <div class="forecast-container">
      <div v-if="weatherData" class="weather-now">
        <h2>{{ weatherData.name }}, {{ weatherData.sys.country }}</h2>
        <h3>{{ currentDate }}</h3>
        <div class="status-box">
          <img :src="iconUrl" alt="Weather Icon" class="weather-icon-big" />
          <p class="status-now">{{ temperature }}°C, {{ weatherData.weather[0].description }}</p>
        </div>
      </div>

      <div
        class="forecast-hour-list-item"
        v-for="(forecast, index) in hourlyForecast.slice(0, 4)"
        :key="index"
        @mouseover="setBackgroundClass(forecast.icon)"
        @mouseleave="resetBackgroundClass"
      >
        <h3 class="time">{{ forecast.time }}</h3>
        <div class="status-box">
          <img :src="iconMapping[forecast.icon]" alt="Weather Icon" class="weather-icon" />
          <p class="status-hourly">
            <template v-if="forecast.temp_min === forecast.temp_max">
              {{ forecast.temp_min }}°C, {{ forecast.description }}
            </template>
            <template v-else>
              {{ forecast.temp_min }}°C - {{ forecast.temp_max }}°C, {{ forecast.description }}
            </template>
          </p>
        </div>
      </div>

      <div class="spacer-row"></div>

      <div
        class="forecast-day-list-item"
        v-for="(forecast, index) in dailyForecast"
        :key="index"
        @mouseover="setHoverBackground(forecast.icon)"
        @mouseleave="resetHoverBackground"
      >
        <h2 class="date">{{ forecast.date }}</h2>
        <div class="status-box">
          <img :src="iconMapping[forecast.icon]" alt="Weather Icon" class="weather-icon" />
          <p class="status-hourly">
            <template v-if="forecast.temp_min === forecast.temp_max">
              {{ forecast.temp_min }}°C, {{ forecast.description }}
            </template>
            <template v-else>
              {{ forecast.temp_min }}°C - {{ forecast.temp_max }}°C, {{ forecast.description }}
            </template>
          </p>
        </div>
      </div>
    </div>
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
  if (!city.value) {
    suggestions.value = []
    return
  }
  try {
    const url = `http://api.openweathermap.org/data/2.5/find?q=${city.value}&appid=${apikey}`
    const response = await axios.get(url)
    suggestions.value = response.data.list.map((city) => ({
      name: city.name,
      country: city.sys.country
    }))
  } catch (error) {
    console.error('Error fetching city suggestions:', error)
  }
}
</script>
