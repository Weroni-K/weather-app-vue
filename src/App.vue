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
