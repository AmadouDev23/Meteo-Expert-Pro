<template>
  <div class="map-container">
    <div ref="mapEl" class="leaflet-map"></div>
    <div class="map-overlay">
      <span class="map-label"><i class="ri-global-line"></i> Carte Interactive</span>
      <span class="map-hint">Cliquez sur un marqueur pour voir la météo</span>
    </div>
    <div class="map-legend">
      <span v-for="(meta, key) in continentMeta" :key="key" class="legend-dot" :style="{ background: meta.color }" :title="meta.label"></span>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted, watch } from 'vue'
import 'leaflet/dist/leaflet.css'
import { CONTINENT_META } from '../data/countries.js'

const props = defineProps({
  weatherMap: { type: Object, default: () => ({}) },
  activeContinent: { type: String, default: 'all' },
  countries: { type: Array, required: true }
})
const emit = defineEmits(['select-city'])

const mapEl = ref(null)
const continentMeta = CONTINENT_META
let map = null, L = null, markerGroup = null

const COLORS = {
  europe: '#4facfe', asia: '#ffd166', africa: '#06d6a0',
  americas: '#f093fb', oceania: '#ff9966'
}

const makeIcon = (country) => {
  const color = COLORS[country.continent] || '#4facfe'
  const w = props.weatherMap[country.capital]
  const temp = w ? `${Math.round(w.main.temp)}°` : '···'
  const img = w ? `<img src="https://openweathermap.org/img/wn/${w.weather[0].icon}.png" class="mk-img">` : ''
  return L.divIcon({
    className: '',
    html: `<div class="mk-pill" style="--ac:${color}">${country.flag}${img}<b>${temp}</b></div>`,
    iconSize: [0, 0],
    iconAnchor: [0, 0]
  })
}

const rebuild = () => {
  if (!L || !map || !markerGroup) return
  markerGroup.clearLayers()
  const list = props.activeContinent === 'all'
    ? props.countries
    : props.countries.filter(c => c.continent === props.activeContinent)
  list.forEach(c => {
    const mk = L.marker([c.lat, c.lon], { icon: makeIcon(c) })
    mk.bindTooltip(
      `<b>${c.flag} ${c.country}</b><br><small>${c.capital}</small>`,
      { className: 'mk-tip', direction: 'top', offset: [0, -5] }
    )
    mk.on('click', () => emit('select-city', c.capital))
    markerGroup.addLayer(mk)
  })
}

onMounted(async () => {
  const lf = await import('leaflet')
  L = lf.default || lf
  map = L.map(mapEl.value, {
    center: [22, 10], zoom: 2, minZoom: 1.5,
    scrollWheelZoom: true, attributionControl: false
  })
  L.tileLayer('https://{s}.basemaps.cartocdn.com/dark_nolabels/{z}/{x}/{y}{r}.png', {
    subdomains: 'abcd', maxZoom: 19
  }).addTo(map)
  L.tileLayer('https://{s}.basemaps.cartocdn.com/dark_only_labels/{z}/{x}/{y}{r}.png', {
    subdomains: 'abcd', opacity: 0.55
  }).addTo(map)
  markerGroup = L.layerGroup().addTo(map)
  rebuild()
})
onUnmounted(() => { if (map) { map.remove(); map = null } })
watch(() => props.weatherMap, rebuild, { deep: true })
watch(() => props.activeContinent, rebuild)
</script>

<!-- Global styles for Leaflet markers (must NOT be scoped) -->
<style>
.mk-pill {
  display: flex; align-items: center; gap: 3px;
  padding: 3px 9px 3px 6px;
  background: rgba(5, 10, 28, 0.92);
  border: 1.5px solid var(--ac, #4facfe);
  border-radius: 20px; color: white;
  font-size: 0.73rem; font-weight: 700;
  white-space: nowrap; cursor: pointer;
  transform: translate(-50%, -50%);
  box-shadow: 0 0 12px rgba(79,172,254,0.2);
  transition: all 0.15s ease;
  backdrop-filter: blur(8px);
  font-family: 'Outfit', sans-serif;
}
.mk-pill:hover {
  box-shadow: 0 0 22px var(--ac, #4facfe);
  border-color: white;
  transform: translate(-50%, -50%) scale(1.15);
  z-index: 9999 !important;
}
.mk-img { width: 20px; height: 20px; }
.mk-tip {
  background: rgba(5,10,28,0.95) !important;
  border: 1px solid rgba(255,255,255,0.15) !important;
  color: white !important;
  border-radius: 10px !important;
  padding: 8px 12px !important;
  font-family: 'Outfit', sans-serif !important;
  box-shadow: 0 10px 30px rgba(0,0,0,0.5) !important;
  font-size: 0.85rem !important;
}
.mk-tip::before { border-top-color: rgba(5,10,28,0.95) !important; }
</style>

<style scoped>
.map-container {
  position: relative;
  border-radius: 24px;
  overflow: hidden;
  border: 1px solid rgba(255,255,255,0.12);
  box-shadow: 0 25px 60px rgba(0,0,0,0.4), inset 0 0 0 1px rgba(255,255,255,0.05);
  height: 420px;
  margin-bottom: 30px;
}
.leaflet-map { width: 100%; height: 100%; }

.map-overlay {
  position: absolute; top: 16px; left: 50%;
  transform: translateX(-50%); z-index: 1000;
  display: flex; align-items: center; gap: 12px;
  background: rgba(5,10,28,0.82);
  border: 1px solid rgba(255,255,255,0.12);
  border-radius: 30px; padding: 7px 18px;
  backdrop-filter: blur(12px);
  pointer-events: none; white-space: nowrap;
}
.map-label { font-weight: 700; color: #4facfe; font-size: 0.85rem; }
.map-label i { margin-right: 5px; }
.map-hint  { font-size: 0.73rem; opacity: 0.55; color: white; }

.map-legend {
  position: absolute; bottom: 14px; right: 14px; z-index: 1000;
  display: flex; gap: 6px; align-items: center;
  background: rgba(5,10,28,0.7);
  border: 1px solid rgba(255,255,255,0.1);
  border-radius: 20px; padding: 6px 12px;
  backdrop-filter: blur(10px);
}
.legend-dot {
  width: 10px; height: 10px; border-radius: 50%;
  opacity: 0.9; cursor: default;
}
</style>
