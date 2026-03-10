<script setup>
import { ref, computed, onMounted, watch } from 'vue'
import MainDashboard from './components/MainDashboard.vue'
import TemperatureChart from './components/TemperatureChart.vue'
import ForecastGrid from './components/ForecastGrid.vue'
import AirQualityWidget from './components/AirQualityWidget.vue'
import UvWidget from './components/UvWidget.vue'

// --- ETAT (STATE) ---
const city = ref('')
const weather = ref(null)
const forecast = ref([])
const aqiData = ref(null)
const uvIndex = ref(null)
const error = ref(null)
const loading = ref(false)
const recentSearches = ref([])

// Nouveautés Premium
const unit = ref('metric')
const isMobileMenuOpen = ref(false)

// --- TA CLÉ API ICI (sécurisée dans .env.local) ---
const API_KEY = import.meta.env.VITE_OPENWEATHER_API_KEY

const toggleUnit = () => {
  unit.value = unit.value === 'metric' ? 'imperial' : 'metric';
  if (weather.value) {
    // Re-fetch data for the current city to update units
    const { lat, lon } = weather.value.coord;
    fetchWeather(`lat=${lat}&lon=${lon}`);
  }
}

const toggleMobileMenu = () => {
  isMobileMenuOpen.value = !isMobileMenuOpen.value;
}

// --- FONCTIONS ET COMPUTED ---
const bgClass = computed(() => {
  if (!weather.value) return '';
  const main = weather.value.weather[0].main;
  if (['Rain', 'Drizzle', 'Thunderstorm'].includes(main)) return 'rainy';
  if (main === 'Clouds') return 'cloudy';
  if (main === 'Snow') return 'snowy';
  return 'sunny';
});

watch(bgClass, (newClass) => {
  document.body.className = newClass;
});

const loadRecentSearches = () => {
  const saved = localStorage.getItem('meteo-recent-searches');
  if (saved) {
    recentSearches.value = JSON.parse(saved);
  }
}

const saveRecentSearch = (cityName) => {
  if (!cityName) return;
  recentSearches.value = recentSearches.value.filter(c => c.toLowerCase() !== cityName.toLowerCase());
  recentSearches.value.unshift(cityName);
  if (recentSearches.value.length > 5) recentSearches.value.pop();
  localStorage.setItem('meteo-recent-searches', JSON.stringify(recentSearches.value));
}

const clearRecentSearches = () => {
  recentSearches.value = [];
  localStorage.removeItem('meteo-recent-searches');
}

const fetchWeather = async (params, isSearch = false) => {
  error.value = null; 
  loading.value = true; 
  weather.value = null; 
  forecast.value = [];
  aqiData.value = null;

  try {
    // 1. Current Weather
    const resW = await fetch(`https://api.openweathermap.org/data/2.5/weather?${params}&units=${unit.value}&lang=fr&appid=${API_KEY}`);
    if (!resW.ok) throw new Error('Ville introuvable');
    const wData = await resW.json();
    weather.value = wData;

    if (isSearch && weather.value.name) {
      saveRecentSearch(weather.value.name);
    }
    
    isMobileMenuOpen.value = false; // Fermer le menu sur succès

    const { lat, lon } = wData.coord;

    // 2. Forecast (5 days / 3 hours)
    const resF = await fetch(`https://api.openweathermap.org/data/2.5/forecast?lat=${lat}&lon=${lon}&units=${unit.value}&lang=fr&appid=${API_KEY}`);
    if (resF.ok) {
      const dataF = await resF.json();
      forecast.value = dataF.list; // On garde toute la liste pour Hourly. ForecastGrid filtrera pour les jours.
    }

    // 3. Air Pollution
    const resAqi = await fetch(`https://api.openweathermap.org/data/2.5/air_pollution?lat=${lat}&lon=${lon}&appid=${API_KEY}`);
    if (resAqi.ok) {
      aqiData.value = await resAqi.json();
    }

    // 4. OneCall pour UV (OpenWeather gratuit ne donne l'UV que via endpoints spécifiques, on mock si non dispo ou on utilise un endpoint 2.5/uvi)
    const resUv = await fetch(`https://api.openweathermap.org/data/2.5/uvi?lat=${lat}&lon=${lon}&appid=${API_KEY}`);
    if (resUv.ok) {
      const uvDataObj = await resUv.json();
      uvIndex.value = uvDataObj.value;
    } else {
      // Mock si l'API UV n'est pas accessible sur le compte (arrive souvent)
      uvIndex.value = Math.random() * 8 + 1; // Fake UV
    }
    
    city.value = '';
  } catch (e) { 
    error.value = e.message; 
  } finally { 
    loading.value = false; 
  }
}

const getWeatherByCity = () => { 
  if(city.value) fetchWeather(`q=${city.value}`, true); 
}

const searchRecent = (recentCity) => {
  city.value = recentCity;
  getWeatherByCity();
}

const getUserLocation = () => {
  if (navigator.geolocation) {
    loading.value = true;
    navigator.geolocation.getCurrentPosition(
      pos => fetchWeather(`lat=${pos.coords.latitude}&lon=${pos.coords.longitude}`),
      () => { 
        error.value = "Géolocalisation refusée"; 
        loading.value = false; 
      }
    );
  }
}

onMounted(() => {
  loadRecentSearches();
  if (recentSearches.value.length === 0) {
    if (navigator.geolocation) {
      loading.value = true;
      navigator.geolocation.getCurrentPosition(
        pos => fetchWeather(`lat=${pos.coords.latitude}&lon=${pos.coords.longitude}`),
        () => { loading.value = false; }
      );
    }
  } else {
    searchRecent(recentSearches.value[0]);
  }
});

const dailyForecast = computed(() => {
  if(!forecast.value) return [];
  // Filtrer pour n'avoir qu'une prévision par jour (vers 12h)
  return forecast.value.filter(x => x.dt_txt.includes("12:00:00"));
});
</script>

<template>
  <div class="dashboard-layout" :class="bgClass">
    
    <!-- Bouton Hamburger (Uniquement sur mobile) -->
    <button class="hamburger-btn" @click="toggleMobileMenu">
      <i class="ri-menu-4-line" v-if="!isMobileMenuOpen"></i>
      <i class="ri-close-line" v-else></i>
    </button>

    <!-- SIDEBAR GAUCHE -->
    <aside class="sidebar" :class="{ 'mobile-open': isMobileMenuOpen }">
      <div class="sidebar-top-bar">
        <div class="logo">
          <i class="ri-blaze-fill"></i>
          <span>MétéoPro</span>
        </div>
        
        <button class="unit-toggle" @click="toggleUnit">
          <span :class="{ active: unit === 'metric' }">°C</span>
          <span :class="{ active: unit === 'imperial' }">°F</span>
        </button>
      </div>

      <div class="search-box">
        <input 
          v-model="city" 
          @keyup.enter="getWeatherByCity" 
          type="text" 
          placeholder="Rechercher une ville..."
        >
        <button class="btn-geo" @click="getUserLocation" title="Ma position"><i class="ri-map-pin-user-line"></i></button>
        <button class="btn-search" @click="getWeatherByCity" title="Rechercher"><i class="ri-search-line"></i></button>
      </div>

      <!-- Recherches Récents -->
      <div v-if="recentSearches.length > 0" class="recent-searches">
        <div class="recent-header">
          <span>Recherches récentes</span>
          <button class="btn-clear" @click="clearRecentSearches"><i class="ri-delete-bin-line"></i></button>
        </div>
        <div class="recent-tags">
          <span 
            v-for="recent in recentSearches" 
            :key="recent" 
            class="recent-tag"
            @click="searchRecent(recent)"
          >
            <i class="ri-history-line"></i> {{ recent }}
          </span>
        </div>
      </div>
    </aside>

    <!-- MAIN CONTENT DROITE -->
    <main class="main-content">
      
      <!-- Toast pour les erreurs -->
      <transition name="toast">
        <div v-if="error" class="error-toast">
          <i class="ri-error-warning-fill"></i> {{ error }}
        </div>
      </transition>

      <!-- Skeletons (Loading state) -->
      <div v-if="loading" class="skeleton-dashboard">
        <div class="skeleton skeleton-hero"></div>
        <div class="skeleton-row">
           <div class="skeleton skeleton-widget" v-for="i in 4" :key="i"></div>
        </div>
      </div>

      <!-- Données Météo -->
      <div v-else-if="weather" class="dashboard-data">
        <!-- Haut: Météo Principale -->
        <MainDashboard :weather="weather" />
        
        <!-- Milieu/Bas : Layout 2 colonnes sur Desktop -->
        <div class="bottom-grid">
          <div class="left-col">
            <TemperatureChart v-if="forecast.length > 0" :forecast="forecast" />
            <ForecastGrid v-if="dailyForecast.length > 0" :forecast="dailyForecast" />
          </div>
          <div class="right-col premium-widgets">
            <AirQualityWidget v-if="aqiData" :aqiData="aqiData" />
            <UvWidget v-if="uvIndex !== null" :uvIndex="uvIndex" />
          </div>
        </div>
      </div>

      <div v-else class="waiting-state">
        <i class="ri-dashboard-3-line"></i>
        <p>Dashboard en attente de données...</p>
      </div>

    </main>
  </div>
</template>

<style scoped>
/* --- LAYOUT GLOBAL (DASHBOARD) --- */
.dashboard-layout {
  display: flex;
  width: 100%;
  max-width: 1400px;
  min-height: 90vh;
  margin: 0 auto;
  gap: 20px;
  transition: all 0.5s ease;
}

@media (max-width: 900px) {
  .dashboard-layout {
    flex-direction: column;
    height: auto;
  }
}

/* Local context styles for dark modes (rain/clouds) */
.dashboard-layout.rainy, .dashboard-layout.cloudy {
  color: #fff; 
  --widget-bg: rgba(0,0,0,0.3);
  --hero-bg: rgba(0,0,0,0.4);
  --temp-color1: #fff;
  --temp-color2: #ccc;
}
.dashboard-layout.rainy input, .dashboard-layout.cloudy input { color: #fff; }
.dashboard-layout.rainy .search-box, .dashboard-layout.cloudy .search-box { background: rgba(255,255,255,0.15); }
.dashboard-layout.rainy .recent-tag, .dashboard-layout.cloudy .recent-tag { background: rgba(0,0,0,0.3); }

/* --- SIDEBAR --- */
.sidebar {
  width: 320px;
  background: var(--widget-bg, rgba(255, 255, 255, 0.4));
  border-radius: 30px;
  padding: 30px;
  backdrop-filter: blur(15px); 
  -webkit-backdrop-filter: blur(15px);
  border: 1px solid rgba(255, 255, 255, 0.4);
  display: flex;
  flex-direction: column;
  box-shadow: 0 10px 40px rgba(0,0,0,0.05);
}

@media (max-width: 900px) {
  .sidebar { 
    position: fixed;
    top: 0;
    left: -100%; /* Cachée par défaut */
    height: 100vh;
    border-radius: 0;
    z-index: 2000; /* Au-dessus de tout */
    padding-top: 80px;
    background: rgba(40,40,40,0.95); /* Plus opaque sur mobile pour masquer le contenu */
    border-right: 1px solid rgba(255,255,255,0.2);
    box-shadow: 10px 0 30px rgba(0,0,0,0.4);
    transition: left 0.4s cubic-bezier(0.175, 0.885, 0.32, 1.275);
  }
  
  .dashboard-layout.rainy .sidebar, .dashboard-layout.cloudy .sidebar {
    background: rgba(10,10,10,0.95);
  }

  .sidebar.mobile-open {
    left: 0; /* Tiroir ouvert */
  }
}

.sidebar-top-bar {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 40px;
}

.logo {
  display: flex;
  align-items: center;
  gap: 10px;
  font-size: 1.8rem;
  font-weight: 800;
  letter-spacing: -1px;
}
.logo i { color: #4facfe; }

/* --- HAMBURGER & UNIT TOGGLE --- */
.hamburger-btn {
  display: none;
  position: absolute;
  top: 20px;
  left: 20px;
  width: 50px;
  height: 50px;
  background: var(--hero-bg, rgba(255, 255, 255, 0.8));
  border: 1px solid rgba(255,255,255,0.3);
  z-index: 2001;
  color: #333;
}

.dashboard-layout.rainy .hamburger-btn, .dashboard-layout.cloudy .hamburger-btn {
  color: #fff;
}

@media (max-width: 900px) {
  .hamburger-btn { display: flex; }
  .dashboard-layout { padding-top: 60px; } /* Eviter que le contenu remonte sous le hamburger */
}

.unit-toggle {
  display: flex;
  background: rgba(0,0,0,0.1);
  border-radius: 30px;
  width: 80px;
  height: 35px;
  padding: 4px;
  position: relative;
}

.dashboard-layout.rainy .unit-toggle, .dashboard-layout.cloudy .unit-toggle {
  background: rgba(255,255,255,0.2);
}

.unit-toggle span {
  flex: 1;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 0.9rem;
  font-weight: 700;
  z-index: 2;
  transition: color 0.3s;
  color: inherit;
  opacity: 0.5;
}

.unit-toggle span.active {
  opacity: 1;
}

.unit-toggle::before {
  content: '';
  position: absolute;
  top: 4px;
  bottom: 4px;
  left: 4px;
  width: calc(50% - 4px);
  background: #fff;
  border-radius: 20px;
  box-shadow: 0 2px 5px rgba(0,0,0,0.1);
  transition: transform 0.3s cubic-bezier(0.175, 0.885, 0.32, 1.275);
}

.dashboard-layout.rainy .unit-toggle::before, .dashboard-layout.cloudy .unit-toggle::before {
  background: rgba(0,0,0,0.3);
}

.unit-toggle:has(span:nth-child(2).active)::before {
  transform: translateX(100%);
}

/* --- MAIN CONTENT --- */
.main-content {
  flex: 1;
  display: flex;
  flex-direction: column;
  position: relative;
}

.dashboard-data {
  display: flex;
  flex-direction: column;
  height: 100%;
}

.bottom-grid {
  display: grid;
  grid-template-columns: 2fr 1fr;
  gap: 20px;
}

.premium-widgets {
  display: flex;
  flex-direction: column;
  gap: 20px;
}

@media (max-width: 1024px) {
  .bottom-grid { grid-template-columns: 1fr; }
}

/* --- RECHERCHE (Sidebar) --- */
.search-box {
  display: flex; 
  gap: 8px; 
  margin-bottom: 30px;
  background: rgba(255,255,255,0.8); 
  padding: 8px; 
  border-radius: 50px;
  box-shadow: 0 4px 15px rgba(0,0,0,0.05);
}

input { 
  flex: 1; min-width: 0; padding: 10px 15px; border: none; background: transparent; outline: none; font-size: 1rem; color: #333; 
}

button {
  width: 45px; height: 45px; flex-shrink: 0; border: none; border-radius: 50%; color: white; cursor: pointer; display: flex; align-items: center; justify-content: center; transition: 0.2s; font-size: 1.2rem;
}
button:hover { transform: scale(1.1); }
.btn-search { background: linear-gradient(135deg, #4facfe, #00f2fe); }
.btn-geo { background: linear-gradient(135deg, #43e97b, #38f9d7); }

/* --- RECHERCHES RECENTES (Sidebar) --- */
.recent-header {
  display: flex; justify-content: space-between; align-items: center; font-size: 0.9rem; font-weight: 600; margin-bottom: 15px; opacity: 0.8;
}
.btn-clear { width: 30px; height: 30px; background: rgba(0,0,0,0.05); color: inherit; border-radius: 8px; }
.btn-clear:hover { background: #ff7675; color: white; }

.recent-tags { display: flex; flex-direction: column; gap: 10px; }
.recent-tag {
  background: rgba(255,255,255,0.5); padding: 12px 15px; border-radius: 15px; font-weight: 500; cursor: pointer; transition: all 0.2s; display: flex; align-items: center; gap: 10px; border: 1px solid rgba(255,255,255,0.3); font-size: 0.95rem;
}
.recent-tag:hover { transform: translateX(5px); background: rgba(255,255,255,0.8); }

/* --- SKELETONS --- */
.skeleton-dashboard { display: flex; flex-direction: column; gap: 20px; width: 100%; animation: fadeIn 0.3s ease-in; }
.skeleton { background: linear-gradient(90deg, rgba(255,255,255,0.3) 25%, rgba(255,255,255,0.6) 50%, rgba(255,255,255,0.3) 75%); background-size: 200% 100%; animation: skeleton-loading 1.5s infinite linear; border-radius: 30px; }
.dashboard-layout.rainy .skeleton, .dashboard-layout.cloudy .skeleton { background: linear-gradient(90deg, rgba(0,0,0,0.1) 25%, rgba(0,0,0,0.2) 50%, rgba(0,0,0,0.1) 75%); background-size: 200% 100%; }
.skeleton-hero { height: 350px; width: 100%; }
.skeleton-row { display: grid; grid-template-columns: repeat(4, 1fr); gap: 20px; }
.skeleton-widget { height: 150px; border-radius: 20px;}

@media (max-width: 1024px) { .skeleton-row { grid-template-columns: repeat(2, 1fr); } }

@keyframes skeleton-loading { 0% { background-position: 200% 0; } 100% { background-position: -200% 0; } }

/* --- TOAST --- */
.error-toast { position: absolute; top: 0; left: 50%; transform: translateX(-50%); background: #ff7675; color: white; padding: 15px 30px; border-radius: 50px; box-shadow: 0 10px 25px rgba(255, 118, 117, 0.4); display: flex; align-items: center; gap: 10px; font-weight: 600; z-index: 1000; }
.toast-enter-active, .toast-leave-active { transition: all 0.4s cubic-bezier(0.175, 0.885, 0.32, 1.275); }
.toast-enter-from, .toast-leave-to { opacity: 0; transform: translate(-50%, -30px) scale(0.9); }

/* --- ETATS --- */
.waiting-state { display: flex; flex-direction: column; align-items: center; justify-content: center; height: 100%; opacity: 0.5; }
.waiting-state i { font-size: 5rem; margin-bottom: 20px;}

@keyframes fadeIn { from { opacity: 0; transform: translateY(15px); } to { opacity: 1; transform: translateY(0); } }
</style>