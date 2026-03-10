<script setup>
import { computed } from 'vue'

const props = defineProps({
  forecast: {
    type: Array,
    required: true
  }
})

const getDayName = (timestamp) => {
  return new Date(timestamp * 1000).toLocaleDateString('fr-FR', { weekday: 'short' });
}

const getWeatherIcon = (code) => {
  const map = {
    '01d': 'ri-sun-fill', '01n': 'ri-moon-clear-fill',
    '02d': 'ri-sun-cloudy-fill', '02n': 'ri-moon-cloudy-fill',
    '03d': 'ri-cloudy-fill', '03n': 'ri-cloudy-fill',
    '04d': 'ri-cloudy-fill', '04n': 'ri-cloudy-fill',
    '09d': 'ri-showers-fill', '09n': 'ri-showers-fill',
    '10d': 'ri-rainy-fill', '10n': 'ri-rainy-fill',
    '11d': 'ri-thunderstorms-fill', '11n': 'ri-thunderstorms-fill',
    '13d': 'ri-snowy-fill', '13n': 'ri-snowy-fill',
    '50d': 'ri-mist-fill', '50n': 'ri-mist-fill'
  };
  return map[code] || 'ri-sun-cloudy-fill';
}
</script>

<template>
  <div v-if="forecast.length > 0" class="forecast-wrapper">
    <div class="forecast-title">Prévisions 5 jours</div>
    <div class="forecast-container">
      <div v-for="(day, index) in forecast" :key="index" class="forecast-card">
        <div class="f-day">{{ getDayName(day.dt) }}</div>
        <i :class="['f-icon', getWeatherIcon(day.weather[0].icon)]"></i>
        <div class="f-temp">{{ Math.round(day.main.temp) }}°</div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.forecast-wrapper {
  text-align: left;
}

.forecast-title {
  font-size: 0.9rem;
  font-weight: 600;
  margin-bottom: 10px;
  opacity: 0.8;
}

.forecast-container {
  display: flex;
  overflow-x: auto;
  gap: 10px;
  padding-bottom: 5px;
}

.forecast-card {
  background: var(--card-bg, rgba(255, 255, 255, 0.4));
  border-radius: 15px;
  padding: 10px;
  flex: 1;
  min-width: 70px;
  display: flex;
  flex-direction: column;
  align-items: center;
  border: 1px solid rgba(255, 255, 255, 0.2);
}

.f-day {
  font-size: 0.8rem;
  font-weight: 600;
}

.f-icon {
  font-size: 2rem;
  margin: 5px 0;
  background: -webkit-linear-gradient(var(--temp-color1, #333), var(--temp-color2, #666)); 
  -webkit-background-clip: text; 
  -webkit-text-fill-color: transparent;
}

.f-temp {
  font-weight: 800;
  font-size: 0.9rem;
}

/* Custom scrollbar for webkit browsers */
.forecast-container::-webkit-scrollbar {
  height: 4px;
}
.forecast-container::-webkit-scrollbar-thumb {
  background: rgba(0,0,0,0.2);
  border-radius: 4px;
}
</style>
