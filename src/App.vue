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
import { ref, computed, onMounted } from 'vue'
import axios from 'axios'

const apikey = 'b09588d826e97199e406a30e962cdb60'

const city = ref('')
const weatherData = ref(null)
const hourlyForecast = ref([])
const dailyForecast = ref([])
const suggestions = ref([])

const temperature = computed(() =>
  weatherData.value ? Math.floor(weatherData.value.main.temp - 273) : null
)

const iconMapping = {
  '01d': '/src/assets/icons/rain.svg',
  '01n': '/src/assets/icons/rain.svg',
  '02d': '/src/assets/icons/few-clouds-day.png',
  '02n': '/src/assets/icons/few-clouds-night.png',
  '03d': '/src/assets/icons/scattered-clouds-day.png',
  '03n': '/src/assets/icons/scattered-clouds-night.png',
  '04d': '/src/assets/icons/broken-clouds-day.png',
  '04n': '/src/assets/icons/broken-clouds-night.png',
  '09d': '/src/assets/icons/shower-rain-day.png',
  '09n': '/src/assets/icons/shower-rain-night.png',
  '10d': '/src/assets/icons/rain-day.png',
  '10n': '/src/assets/icons/rain-night.png',
  '11d': '/src/assets/icons/thunderstorm-day.png',
  '11n': '/src/assets/icons/thunderstorm-night.png',
  '13d': '/src/assets/icons/snow-day.png',
  '13n': '/src/assets/icons/snow-night.png',
  '50d': '/src/assets/icons/mist-day.png',
  '50n': '/src/assets/icons/mist-night.png'
}

const iconUrl = computed(() =>
  weatherData.value ? iconMapping[weatherData.value.weather[0].icon] : null
)

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

// State for hover
const hoverBackgroundClass = ref('')

// Method to update the background on hover
const setHoverBackground = (icon) => {
  hoverBackgroundClass.value = getBackgroundClass(icon)
}

// Method to reset the background when not hovering
const resetHoverBackground = () => {
  hoverBackgroundClass.value = ''
}

const getBackgroundClass = (icon) => {
  return backgroundMapping[icon] || ''
}

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

const selectCity = (selectedCity) => {
  city.value = `${selectedCity.name}, ${selectedCity.country}`
  searchByCity()
  suggestions.value = []
}

const fetchCurrentLocationWeather = async () => {
  if (navigator.geolocation) {
    navigator.geolocation.getCurrentPosition(async (position) => {
      const { latitude, longitude } = position.coords
      const url = `http://api.openweathermap.org/data/2.5/weather?lat=${latitude}&lon=${longitude}&appid=${apikey}`
      await fetchWeatherData(url)
    })
  }
}
// other data: const url = `https://api.openweathermap.org/data/3.0/onecall?lat=${latitude}&lon=${longitude}&appid=${apikey}`
const fetchWeatherData = async (url) => {
  try {
    const response = await axios.get(url)
    weatherData.value = response.data
    // Fetch forecast data
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
    // Fetch forecast data
    await fetchForecast(weatherData.value.name)
  } catch (error) {
    console.error('Error fetching weather data:', error)
  }
  city.value = ''
}

const hourForecast = (forecast) => {
  hourlyForecast.value = []
  for (let i = 0; i < 5; i++) {
    const date = new Date(forecast.list[i].dt * 1000)
    hourlyForecast.value.push({
      time: date.toLocaleTimeString([], { hour: '2-digit', minute: '2-digit', hour12: false }), // 24-hour format
      temp_max: Math.floor(forecast.list[i].main.temp_max - 273),
      temp_min: Math.floor(forecast.list[i].main.temp_min - 273),
      description: forecast.list[i].weather[0].description,
      icon: forecast.list[i].weather[0].icon
    })
  }
}

const dayForecast = (forecast) => {
  dailyForecast.value = []
  for (let i = 8; i < forecast.list.length; i += 8) {
    const date = new Date(forecast.list[i].dt * 1000)
    dailyForecast.value.push({
      date: date.toLocaleDateString('en-GB', {
        weekday: 'short', // "Wed"
        day: 'numeric', // "18"
        month: 'short' // "Sep"
      }),
      temp_max: Math.floor(forecast.list[i].main.temp_max - 273),
      temp_min: Math.floor(forecast.list[i].main.temp_min - 273),
      description: forecast.list[i].weather[0].description,
      icon: forecast.list[i].weather[0].icon
    })
  }
}
const currentDate = computed(() => {
  const options = { weekday: 'short', day: 'numeric', month: 'short' }
  return new Date().toLocaleDateString('en-GB', options)
})

onMounted(fetchCurrentLocationWeather)
</script>

<style scoped>
.sunny-day {
  background: url('/src/assets/sunny-day.jpg') no-repeat center center;
  background-size: cover;
}

.sunny-night {
  background: url('/src/assets/sunny-night.png') no-repeat center center;
  background-size: cover;
}

.few-clouds-day {
  background: url('/src/assets/clouds.jpg') no-repeat center center;
  background-size: cover;
}

.few-clouds-night {
  background: url('/src/assets/few-clouds-night.jpg') no-repeat center center;
  background-size: cover;
}

.scattered-clouds-day {
  background: url('/src/assets/clouds.jpg') no-repeat center center;
  background-size: cover;
}

.scattered-clouds-night {
  background: url('/src/assets/scattered-clouds-night.jpg') no-repeat center center;
  background-size: cover;
}

.broken-clouds-day {
  background: url('/src/assets/broken-clouds-day.jpg') no-repeat center center;
  background-size: cover;
}

.broken-clouds-night {
  background: url('/src/assets/broken-clouds-night.jpg') no-repeat center center;
  background-size: cover;
}

.shower-rain-day {
  background: url('/src/assets/rain.jpg') no-repeat center center;
  background-size: cover;
}

.shower-rain-night {
  background: url('/src/assets/shower-rain-night.jpg') no-repeat center center;
  background-size: cover;
}

.rain-day {
  background: url('/src/assets/rain.png') no-repeat center center;
  background-size: cover;
}

.rain-night {
  background: url('/src/assets/rain-night.jpg') no-repeat center center;
  background-size: cover;
}

.thunderstorm-day {
  background: url('/src/assets/thunderstorm-day.jpg') no-repeat center center;
  background-size: cover;
}

.thunderstorm-night {
  background: url('/src/assets/thunderstorm-night.jpg') no-repeat center center;
  background-size: cover;
}

.snow-day {
  background: url('/src/assets/snow-day.jpg') no-repeat center center;
  background-size: cover;
}

.snow-night {
  background: url('/src/assets/snow-night.jpg') no-repeat center center;
  background-size: cover;
}

.mist-day {
  background: url('/src/assets/mist-day.jpg') no-repeat center center;
  background-size: cover;
}

.mist-night {
  background: url('/src/assets/mist-night.jpg') no-repeat center center;
  background-size: cover;
}
</style>
