<script setup>
import { computed } from 'vue'

const props = defineProps({
  weather: {
    type: Object,
    required: true
  }
})

const todayDate = computed(() => {
  return new Date().toLocaleDateString('fr-FR', { weekday: 'long', day: 'numeric', month: 'long' });
});

const formatTime = (timestamp) => {
  const date = new Date(timestamp * 1000);
  return date.toLocaleTimeString('fr-FR', { hour: '2-digit', minute: '2-digit' });
}

const getWindDirection = (deg) => {
  const directions = ['N', 'NE', 'E', 'SE', 'S', 'SO', 'O', 'NO'];
  const index = Math.round(((deg %= 360) < 0 ? deg + 360 : deg) / 45) % 8;
  return directions[index];
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
  <div class="main-dashboard-content">
    
    <!-- HEADER HERO: Grande section du haut -->
    <div class="hero-section">
      <div class="hero-info">
        <h1 class="location">{{ weather.name }}, {{ weather.sys.country }}</h1>
        <p class="date">{{ todayDate }}</p>
        
        <div class="hero-weather">
          <div class="temp">{{ Math.round(weather.main.temp) }}°</div>
          <div class="desc-box">
             <i :class="getWeatherIcon(weather.weather[0].icon)" class="desc-icon"></i>
             <div class="desc-text">{{ weather.weather[0].description }}</div>
          </div>
        </div>
      </div>
      
      <div class="hero-graphic">
        <i :class="getWeatherIcon(weather.weather[0].icon)" class="main-icon"></i>
      </div>
    </div>

    <!-- METRICS GRID -->
    <div class="metrics-grid">
      <!-- Humidité -->
      <div class="metric-card">
        <div class="metric-header">
           <i class="ri-drop-line"></i> Humidité
        </div>
        <div class="metric-value">{{ weather.main.humidity }}<span class="unit">%</span></div>
        <div class="metric-sub">Ressenti {{ Math.round(weather.main.feels_like) }}°</div>
      </div>
      
      <!-- Vent -->
      <div class="metric-card">
        <div class="metric-header">
           <i class="ri-windy-line"></i> Vent
        </div>
        <div class="metric-value">{{ weather.wind.speed }}<span class="unit"> km/h</span></div>
        <div class="metric-sub">Direction {{ getWindDirection(weather.wind.deg) }}</div>
      </div>
      
      <!-- Soleil -->
      <div class="metric-card">
        <div class="metric-header">
           <i class="ri-sun-line"></i> Soleil
        </div>
        <div class="sun-times">
          <div class="sun-time"><i class="ri-sunrise-line"></i> {{ formatTime(weather.sys.sunrise) }}</div>
          <div class="sun-time"><i class="ri-sunset-line"></i> {{ formatTime(weather.sys.sunset) }}</div>
        </div>
      </div>
      
      <!-- Divers (Pression/Nuages) -->
      <div class="metric-card">
        <div class="metric-header">
           <i class="ri-dashboard-2-line"></i> Conditions
        </div>
        <div class="metric-small">Pression: <strong>{{ weather.main.pressure }} hPa</strong></div>
        <div class="metric-small">Nuages: <strong>{{ weather.clouds.all }}%</strong></div>
      </div>
    </div>
    
  </div>
</template>

<style scoped>
.main-dashboard-content { 
  animation: fadeIn 0.5s ease-out; 
}

/* --- HERO SECTION --- */
.hero-section {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 30px;
  background: var(--hero-bg, rgba(255, 255, 255, 0.4));
  border-radius: 30px;
  margin-bottom: 25px;
  border: 1px solid rgba(255,255,255,0.3);
  box-shadow: 0 10px 30px rgba(0,0,0,0.05);
}

.hero-info {
  text-align: left;
}

.location {
  font-size: 2.5rem;
  font-weight: 800;
  margin-bottom: 5px;
  letter-spacing: -1px;
}

.date {
  font-size: 1rem;
  opacity: 0.8;
  margin-bottom: 20px;
  text-transform: capitalize;
}

.hero-weather {
  display: flex;
  align-items: flex-end;
  gap: 20px;
}

.temp {
  font-size: 6rem;
  font-weight: 900;
  line-height: 0.8;
  background: -webkit-linear-gradient(var(--temp-color1, #333), var(--temp-color2, #666)); 
  -webkit-background-clip: text; 
  -webkit-text-fill-color: transparent;
}

.desc-box {
  display: flex;
  align-items: center;
  gap: 8px;
  background: rgba(0,0,0,0.05);
  padding: 8px 15px;
  border-radius: 20px;
}

.body.rainy .desc-box, .body.cloudy .desc-box {
  background: rgba(255,255,255,0.1);
}

.desc-icon {
  font-size: 1.5rem;
}

.desc-text {
  font-size: 1.1rem;
  font-weight: 600;
  text-transform: capitalize;
}

.main-icon {
  font-size: 10rem;
  background: -webkit-linear-gradient(var(--temp-color1, #333), var(--temp-color2, #666)); 
  -webkit-background-clip: text; 
  -webkit-text-fill-color: transparent;
  filter: drop-shadow(0 10px 15px rgba(0,0,0,0.1));
}

@media (max-width: 768px) {
  .hero-section {
    flex-direction: column-reverse;
    text-align: center;
    gap: 20px;
    padding: 20px;
  }
  .hero-info { text-align: center; }
  .hero-weather { justify-content: center; }
  .main-icon { font-size: 6rem; }
}

/* --- METRICS GRID --- */
.metrics-grid {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  gap: 20px;
  margin-bottom: 25px;
}

@media (max-width: 1024px) {
  .metrics-grid { grid-template-columns: repeat(2, 1fr); }
}

@media (max-width: 480px) {
  .metrics-grid { grid-template-columns: 1fr; }
}

.metric-card {
  background: var(--widget-bg, rgba(255, 255, 255, 0.4));
  border-radius: 25px;
  padding: 20px;
  border: 1px solid rgba(255,255,255,0.2);
  display: flex;
  flex-direction: column;
  justify-content: space-between;
}

.metric-header {
  font-size: 0.9rem;
  font-weight: 600;
  opacity: 0.7;
  display: flex;
  align-items: center;
  gap: 5px;
  margin-bottom: 15px;
}

.metric-value {
  font-size: 2.2rem;
  font-weight: 800;
  margin-bottom: 5px;
}

.unit {
  font-size: 1rem;
  font-weight: 600;
}

.metric-sub {
  font-size: 0.85rem;
  opacity: 0.8;
  font-weight: 500;
}

.sun-times {
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.sun-time {
  font-size: 1.1rem;
  font-weight: 700;
  display: flex;
  align-items: center;
  gap: 8px;
}

.metric-small {
  margin-bottom: 8px;
  font-size: 0.95rem;
}

@keyframes fadeIn { 
  from { opacity: 0; transform: translateY(15px); } 
  to { opacity: 1; transform: translateY(0); } 
}
</style>
