<template>
  <div :style="{ backgroundImage: `url(${hoverBackgroundClass})` }" class="container">
    <div class="header">
      <h1>WEATHER APP</h1>
      <div class="header2">
        <div class="search-bar">
          <input
            type="text"
            v-model="city"
            placeholder="City..."
            class="search-input"
            @input="fetchCitySuggestions"
          />
          <span class="span-suggestions"
            ><ul v-if="suggestions.length" class="suggestions-list">
              <li
                v-for="(suggestion, index) in suggestions"
                :key="index"
                @click="selectCity(suggestion)"
                class="suggestion-item"
              >
                {{ suggestion.name }}, {{ suggestion.country }}
              </li>
            </ul>
          </span>
        </div>
        <button @click="searchByCity" class="search-button">Search</button
        ><span class="tooltip">
          <button @click="fetchCurrentLocationWeather" class="location-button">
            <img
              class="location-icon"
              src="/weather-app-vue/src/assets/icons/location.svg"
              alt="Location Icon"
            /></button
          ><span class="tooltip-text">Fetch your current location</span></span
        >
      </div>
    </div>

    <div class="forecast-container">
      <div v-if="weatherData" class="weather-now">
        <div>
          <h2>{{ weatherData.name }}, {{ weatherData.sys.country }}</h2>
          <h3>{{ currentDate }}</h3>
        </div>
        <div class="status-box">
          <img :src="iconUrl" alt="Weather Icon" class="weather-icon-big" />
          <p class="status-now">{{ temperature }}°C, {{ weatherData.weather[0].description }}</p>
        </div>
      </div>

      <div
        class="forecast-hour-list-item"
        v-for="(forecast, index) in hourlyForecast.slice(0, 4)"
        :key="index"
        @mouseover="setHoverBackground(forecast.icon)"
        @mouseleave="resetHoverBackground"
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
  '01d': '/weather-app-vue/src/assets/icons/clear-day.svg',
  '01n': '/weather-app-vue/src/assets/icons/clear-night.svg',
  '02d': '/weather-app-vue/src/assets/icons/clouds-day2.svg', // few-clouds, images downloaded from Freepik
  '02n': '/weather-app-vue/src/assets/icons/clouds-night2.svg',
  '03d': '/weather-app-vue/src/assets/icons/clouds-day1.svg', // scattered-clouds
  '03n': '/weather-app-vue/src/assets/icons/clouds-night1.svg',
  '04d': '/weather-app-vue/src/assets/icons/clouds1.svg', // broken-clouds (overcaast)
  '04n': '/weather-app-vue/src/assets/icons/clouds1.svg',
  '09d': '/weather-app-vue/src/assets/icons/showers.svg',
  '09n': '/weather-app-vue/src/assets/icons/showers.svg',
  '10d': '/weather-app-vue/src/assets/icons/rain.svg',
  '10n': '/weather-app-vue/src/assets/icons/rain.svg',
  '11d': '/weather-app-vue/src/assets/icons/thunderstorm.svg',
  '11n': '/weather-app-vue/src/assets/icons/thunderstorm.svg',
  '13d': '/weather-app-vue/src/assets/icons/snow.svg',
  '13n': '/weather-app-vue/src/assets/icons/snow.svg',
  '50d': '/weather-app-vue/src/assets/icons/mist.svg',
  '50n': '/weather-app-vue/src/assets/icons/mist.svg'
}

const iconUrl = computed(() =>
  weatherData.value ? iconMapping[weatherData.value.weather[0].icon] : null
)

const backgroundMapping = {
  '01d': '/weather-app-vue/src/assets/clear-day.jpg',
  '01n': '/weather-app-vue/src/assets/clear-night.jpg',
  '02d': '/weather-app-vue/src/assets/few-clouds-day.jpg',
  '02n': '/weather-app-vue/src/assets/few-clouds-night.jpg',
  '03d': '/weather-app-vue/src/assets/scattered-clouds-day.jpg',
  '03n': '/weather-app-vue/src/assets/scattered-clouds-night.jpg',
  '04d': '/weather-app-vue/src/assets/broken-clouds-day.jpg',
  '04n': '/weather-app-vue/src/assets/broken-clouds-night.jpg',
  '09d': '/weather-app-vue/src/assets/rain.jpg', //add rain..?
  '09n': '/weather-app-vue/src/assets/rain.jpg', //add rain..?
  '10d': '/weather-app-vue/src/assets/rain.jpg', //add rain..?
  '10n': '/weather-app-vue/src/assets/rain.jpg', //add rain..?
  '11d': '/weather-app-vue/src/assets/thunderstorm-day.jpg',
  '11n': '/weather-app-vue/src/assets/thunderstorm-night.jpg',
  '13d': '/weather-app-vue/src/assets/snow-day.jpg',
  '13n': '/weather-app-vue/src/assets/snow-night.jpg',
  '50d': '/weather-app-vue/src/assets/mist-day.jpg', //add fog..?
  '50n': '/weather-app-vue/src/assets/mist-night.jpg' //add fog..?
}
const currentBackgroundUrl = computed(() =>
  weatherData.value ? backgroundMapping[weatherData.value.weather[0].icon] : ''
)
const hoverBackgroundClass = ref('')

const setHoverBackground = (icon) => {
  hoverBackgroundClass.value = backgroundMapping[icon] || ''
}

const resetHoverBackground = () => {
  hoverBackgroundClass.value = currentBackgroundUrl.value
}

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
