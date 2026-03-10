<script setup>
const props = defineProps({
  forecast: {
    type: Array,
    required: true
  }
})

// Dans OWM, list renvoie par défaut par pas de 3 heures.
// On filtre les 8 premiers pour avoir les prochaines 24h.
const hourlyData = computed(() => {
  return props.forecast.slice(0, 8);
})

const getHour = (timestamp) => {
  const date = new Date(timestamp * 1000);
  return date.toLocaleTimeString('fr-FR', { hour: '2-digit', minute: '2-digit' }).replace(':', 'h');
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

import { computed } from 'vue';
</script>

<template>
  <div class="hourly-widget">
    <div class="widget-header">
      <i class="ri-time-line"></i>
      <span>Prochaines 24h</span>
    </div>
    
    <div class="hourly-container">
      <div v-for="(hour, index) in hourlyData" :key="index" class="hour-item">
        <span class="h-time">{{ index === 0 ? 'Maint.' : getHour(hour.dt) }}</span>
        <i :class="['h-icon', getWeatherIcon(hour.weather[0].icon)]"></i>
        <span class="h-temp">{{ Math.round(hour.main.temp) }}°</span>
      </div>
    </div>
  </div>
</template>

<style scoped>
.hourly-widget {
  background: var(--widget-bg, rgba(255, 255, 255, 0.4));
  border-radius: 20px;
  padding: 20px;
  margin-bottom: 20px;
  border: 1px solid rgba(255,255,255,0.2);
}

.widget-header {
  display: flex;
  align-items: center;
  gap: 8px;
  font-size: 0.9rem;
  font-weight: 600;
  opacity: 0.8;
  margin-bottom: 15px;
}

.hourly-container {
  display: flex;
  overflow-x: auto;
  gap: 15px;
  padding-bottom: 5px;
}

.hourly-container::-webkit-scrollbar {
  height: 4px;
}
.hourly-container::-webkit-scrollbar-thumb {
  background: rgba(0,0,0,0.2);
  border-radius: 4px;
}

.hour-item {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 8px;
  min-width: 60px;
  padding: 10px;
  border-radius: 15px;
  background: rgba(255,255,255,0.2);
  transition: transform 0.2s;
}

.body.rainy .hour-item, .body.cloudy .hour-item {
  background: rgba(0,0,0,0.2);
}

.hour-item:hover {
  transform: translateY(-2px);
  background: rgba(255,255,255,0.4);
}

.h-time {
  font-size: 0.8rem;
  font-weight: 600;
}

.h-icon {
  font-size: 1.5rem;
  background: -webkit-linear-gradient(var(--temp-color1, #333), var(--temp-color2, #666)); 
  -webkit-background-clip: text; 
  -webkit-text-fill-color: transparent;
}

.h-temp {
  font-size: 1rem;
  font-weight: 800;
}
</style>
