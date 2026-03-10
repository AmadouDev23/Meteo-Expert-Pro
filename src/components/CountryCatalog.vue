<template>
  <div class="catalog">

    <!-- HEADER -->
    <div class="catalog-head">
      <div>
        <h2><i class="ri-global-line"></i> Catalogue Mondial</h2>
        <p>Météo en temps réel dans <strong>{{ ALL_COUNTRIES.length }} pays</strong></p>
      </div>
      <div class="head-stats">
        <span class="stat-badge">
          <i :class="loadedCount < ALL_COUNTRIES.length ? 'ri-loader-4-line spin' : 'ri-checkbox-circle-line'"></i>
          {{ loadedCount }} / {{ ALL_COUNTRIES.length }} chargés
        </span>
      </div>
    </div>

    <!-- WORLD MAP -->
    <WorldMap
      :weather-map="weatherMap"
      :active-continent="activeContinent"
      :countries="ALL_COUNTRIES"
      @select-city="onSelect"
    />

    <!-- FILTERS BAR -->
    <div class="filters-bar">
      <div class="continent-tabs">
        <button
          v-for="(meta, key) in CONTINENT_META"
          :key="key"
          class="tab-btn"
          :class="{ active: activeContinent === key }"
          :style="activeContinent === key ? { background: meta.color, borderColor: meta.color } : {}"
          @click="activeContinent = key"
        >
          {{ meta.icon }} {{ meta.label }}
          <span class="tab-count">{{ countByContinent(key) }}</span>
        </button>
      </div>

      <div class="search-mini">
        <i class="ri-search-line"></i>
        <input v-model="search" placeholder="Rechercher un pays ou capitale..." />
        <button v-if="search" class="clear-search" @click="search = ''">
          <i class="ri-close-line"></i>
        </button>
      </div>
    </div>

    <!-- GRID -->
    <div class="countries-grid">
      <transition-group name="card-appear">
        <div
          v-for="c in filteredCountries"
          :key="c.capital"
          class="country-card"
          :class="{ loaded: !!weatherMap[c.capital] }"
          :style="{ '--accent': COLORS[c.continent] || '#4facfe' }"
          @click="onSelect(c.capital)"
        >
          <template v-if="weatherMap[c.capital]">
            <div class="card-flag-wrap">
              <span class="card-flag">{{ c.flag }}</span>
            </div>
            <div class="card-body">
              <div class="card-names">
                <span class="card-country">{{ c.country }}</span>
                <span class="card-capital"><i class="ri-map-pin-2-line"></i> {{ c.capital }}</span>
              </div>
              <div class="card-weather">
                <img
                  :src="`https://openweathermap.org/img/wn/${weatherMap[c.capital].weather[0].icon}@2x.png`"
                  :alt="weatherMap[c.capital].weather[0].description"
                  class="card-icon"
                />
                <span class="card-temp">{{ Math.round(weatherMap[c.capital].main.temp) }}°</span>
              </div>
            </div>
            <div class="card-desc">{{ weatherMap[c.capital].weather[0].description }}</div>
            <div class="card-footer">
              <span><i class="ri-drop-line"></i> {{ weatherMap[c.capital].main.humidity }}%</span>
              <span><i class="ri-wind-line"></i> {{ Math.round(weatherMap[c.capital].wind.speed) }} km/h</span>
              <span class="continent-badge" :style="{ background: COLORS[c.continent] + '30', color: COLORS[c.continent] }">
                {{ CONTINENT_META[c.continent]?.icon }} {{ CONTINENT_META[c.continent]?.label }}
              </span>
            </div>
            <div class="card-glow"></div>
          </template>

          <!-- Skeleton -->
          <template v-else>
            <div class="sk-flag">{{ c.flag }}</div>
            <div class="sk-lines">
              <div class="sk-line sk-l1"></div>
              <div class="sk-line sk-l2"></div>
            </div>
            <div class="sk-temp"></div>
          </template>
        </div>
      </transition-group>
    </div>

    <!-- Empty state -->
    <div v-if="filteredCountries.length === 0" class="empty-state">
      <i class="ri-search-eye-line"></i>
      <p>Aucun pays trouvé pour "<strong>{{ search }}</strong>"</p>
    </div>

  </div>
</template>

<script setup>
import { ref, computed, onMounted, watch, reactive } from 'vue'
import WorldMap from './WorldMap.vue'
import { ALL_COUNTRIES, CONTINENT_META } from '../data/countries.js'

const props = defineProps({
  apiKey: { type: String, required: true },
  unit:   { type: String, default: 'metric' }
})
const emit = defineEmits(['select-city'])

const COLORS = {
  europe: '#4facfe', asia: '#ffd166', africa: '#06d6a0',
  americas: '#f093fb', oceania: '#ff9966'
}

const activeContinent = ref('all')
const search = ref('')
const weatherMap = reactive({})
const loadedCount = ref(0)

const filteredCountries = computed(() => {
  let list = ALL_COUNTRIES
  if (activeContinent.value !== 'all')
    list = list.filter(c => c.continent === activeContinent.value)
  if (search.value.trim()) {
    const q = search.value.toLowerCase().trim()
    list = list.filter(c =>
      c.country.toLowerCase().includes(q) || c.capital.toLowerCase().includes(q)
    )
  }
  return list
})

const countByContinent = (key) => {
  if (key === 'all') return ALL_COUNTRIES.length
  return ALL_COUNTRIES.filter(c => c.continent === key).length
}

const onSelect = (capital) => {
  emit('select-city', capital)
}

const chunk = (arr, n) => Array.from({ length: Math.ceil(arr.length / n) }, (_, i) => arr.slice(i * n, i * n + n))

const fetchAll = async () => {
  const batches = chunk(ALL_COUNTRIES, 8)
  for (const batch of batches) {
    await Promise.all(batch.map(async (c) => {
      try {
        const res = await fetch(
          `https://api.openweathermap.org/data/2.5/weather?lat=${c.lat}&lon=${c.lon}&units=${props.unit}&lang=fr&appid=${props.apiKey}`
        )
        if (res.ok) {
          const data = await res.json()
          weatherMap[c.capital] = data
          loadedCount.value++
        }
      } catch (_) {}
    }))
    await new Promise(r => setTimeout(r, 150))
  }
}

onMounted(fetchAll)
watch(() => props.unit, () => { Object.keys(weatherMap).forEach(k => delete weatherMap[k]); loadedCount.value = 0; fetchAll() })
</script>

<style scoped>
.catalog {
  padding: 30px;
  background: var(--widget-bg, rgba(255,255,255,0.35));
  border-radius: 30px;
  backdrop-filter: blur(20px);
  -webkit-backdrop-filter: blur(20px);
  border: 1px solid rgba(255,255,255,0.3);
  min-height: 600px;
  animation: fadeIn 0.6s ease-out;
}

/* HEADER */
.catalog-head {
  display: flex; justify-content: space-between; align-items: flex-start;
  margin-bottom: 24px;
}
.catalog-head h2 {
  font-size: 1.9rem; font-weight: 800; margin: 0;
  display: flex; align-items: center; gap: 12px;
}
.catalog-head h2 i { color: #4facfe; }
.catalog-head p { opacity: 0.65; font-size: 1rem; margin-top: 4px; }

.stat-badge {
  display: flex; align-items: center; gap: 7px;
  background: rgba(79,172,254,0.12); border: 1px solid rgba(79,172,254,0.3);
  color: #4facfe; border-radius: 20px; padding: 6px 14px;
  font-size: 0.82rem; font-weight: 700;
}
.spin { animation: spin 1s linear infinite; }
@keyframes spin { to { transform: rotate(360deg); } }

/* FILTERS BAR */
.filters-bar {
  display: flex; flex-wrap: wrap; gap: 14px;
  align-items: center; margin-bottom: 24px;
}
.continent-tabs {
  display: flex; flex-wrap: wrap; gap: 8px; flex: 1;
}
.tab-btn {
  height: 38px; padding: 0 14px; border-radius: 20px;
  border: 1.5px solid rgba(255,255,255,0.25);
  background: rgba(255,255,255,0.1); color: inherit;
  font-weight: 600; font-size: 0.82rem; cursor: pointer;
  display: flex; align-items: center; gap: 6px;
  transition: all 0.25s; opacity: 0.75;
}
.tab-btn:hover { opacity: 1; background: rgba(255,255,255,0.2); transform: translateY(-1px); }
.tab-btn.active { color: white; opacity: 1; box-shadow: 0 6px 18px rgba(0,0,0,0.2); }
.tab-count {
  background: rgba(255,255,255,0.25); border-radius: 10px;
  padding: 0 7px; font-size: 0.72rem;
}

.search-mini {
  display: flex; align-items: center; gap: 8px;
  background: rgba(255,255,255,0.2); border: 1.5px solid rgba(255,255,255,0.25);
  border-radius: 20px; padding: 0 14px; min-width: 240px;
}
.search-mini i { opacity: 0.6; font-size: 0.95rem; }
.search-mini input {
  flex: 1; border: none; background: transparent; outline: none;
  font-size: 0.88rem; color: inherit; padding: 9px 0;
}
.clear-search {
  width: 22px; height: 22px; border-radius: 50%;
  background: rgba(0,0,0,0.15); border: none; color: inherit;
  cursor: pointer; display: flex; align-items: center; justify-content: center;
  font-size: 0.8rem; padding: 0;
}

/* GRID */
.countries-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(210px, 1fr));
  gap: 16px;
}

/* CARD */
.country-card {
  position: relative; overflow: hidden;
  padding: 18px; border-radius: 20px;
  background: rgba(255,255,255,0.35);
  border: 1.5px solid rgba(255,255,255,0.2);
  cursor: pointer; transition: all 0.3s cubic-bezier(0.4,0,0.2,1);
  display: flex; flex-direction: column; gap: 10px;
  min-height: 160px;
}
.country-card:hover {
  transform: translateY(-6px);
  border-color: var(--accent, #4facfe);
  box-shadow: 0 20px 40px rgba(0,0,0,0.12);
  background: rgba(255,255,255,0.5);
}
.country-card:hover .card-glow { opacity: 1; }

.card-glow {
  position: absolute; inset: 0; opacity: 0;
  background: radial-gradient(circle at top right, var(--accent, #4facfe) 0%, transparent 70%);
  transition: opacity 0.4s; pointer-events: none;
  mix-blend-mode: overlay;
}

.card-flag-wrap {
  display: flex; justify-content: space-between; align-items: center;
}
.card-flag { font-size: 2rem; line-height: 1; filter: drop-shadow(0 3px 8px rgba(0,0,0,0.2)); }

.card-body {
  display: flex; justify-content: space-between; align-items: center;
}
.card-names { display: flex; flex-direction: column; gap: 2px; }
.card-country { font-size: 1rem; font-weight: 700; }
.card-capital { font-size: 0.72rem; opacity: 0.6; display: flex; align-items: center; gap: 3px; }

.card-weather { display: flex; align-items: center; gap: 2px; }
.card-icon { width: 48px; height: 48px; filter: drop-shadow(0 4px 8px rgba(0,0,0,0.15)); }
.card-temp { font-size: 1.8rem; font-weight: 900; line-height: 1; }

.card-desc {
  font-size: 0.78rem; opacity: 0.65;
  text-transform: capitalize; margin-top: -4px;
}
.card-footer {
  display: flex; flex-wrap: wrap; gap: 6px; align-items: center;
  font-size: 0.72rem; opacity: 0.75; border-top: 1px solid rgba(0,0,0,0.06);
  padding-top: 8px;
}
.continent-badge {
  margin-left: auto; padding: 2px 8px; border-radius: 10px; font-weight: 700; font-size: 0.7rem;
}

/* SKELETON */
.sk-flag { font-size: 2rem; }
.sk-lines { flex: 1; display: flex; flex-direction: column; gap: 8px; margin-top: 8px; }
.sk-line {
  border-radius: 8px; height: 12px;
  background: linear-gradient(90deg, rgba(255,255,255,0.2) 25%, rgba(255,255,255,0.4) 50%, rgba(255,255,255,0.2) 75%);
  background-size: 200% 100%;
  animation: shimmer 1.5s infinite linear;
}
.sk-l1 { width: 70%; }
.sk-l2 { width: 45%; }
.sk-temp {
  align-self: flex-end; width: 55px; height: 30px; border-radius: 10px;
  background: linear-gradient(90deg, rgba(255,255,255,0.2) 25%, rgba(255,255,255,0.4) 50%, rgba(255,255,255,0.2) 75%);
  background-size: 200% 100%;
  animation: shimmer 1.5s infinite linear;
}
@keyframes shimmer { 0%{background-position:200% 0} 100%{background-position:-200% 0} }

/* EMPTY STATE */
.empty-state {
  display: flex; flex-direction: column; align-items: center;
  justify-content: center; padding: 60px 20px; opacity: 0.5;
}
.empty-state i { font-size: 3.5rem; margin-bottom: 15px; }

/* TRANSITIONS */
.card-appear-enter-active { transition: all 0.4s ease; }
.card-appear-enter-from  { opacity: 0; transform: translateY(15px) scale(0.97); }

@keyframes fadeIn { from{opacity:0;transform:translateY(20px)} to{opacity:1;transform:translateY(0)} }

/* DARK MODE (rainy/cloudy parent) */
:global(.rainy) .country-card, :global(.cloudy) .country-card {
  background: rgba(0,0,0,0.3); color: white; border-color: rgba(255,255,255,0.1);
}
:global(.rainy) .catalog, :global(.cloudy) .catalog {
  background: rgba(0,0,0,0.35);
}
:global(.rainy) .search-mini input, :global(.cloudy) .search-mini input { color: white; }

@media (max-width: 700px) {
  .catalog { padding: 20px; }
  .catalog-head { flex-direction: column; gap: 12px; }
  .countries-grid { grid-template-columns: repeat(auto-fill, minmax(160px,1fr)); gap: 12px; }
  .filters-bar { flex-direction: column; align-items: stretch; }
  .search-mini { min-width: unset; }
}
</style>
