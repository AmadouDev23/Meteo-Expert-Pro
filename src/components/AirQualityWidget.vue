<script setup>
import { computed } from 'vue'

const props = defineProps({
  aqiData: {
    type: Object,
    required: true
  }
})

// L'AQI va de 1 (Good) à 5 (Very Poor)
const aqiIndex = computed(() => {
  return props.aqiData?.list?.[0]?.main?.aqi || 1;
})

const getAqiInfo = (index) => {
  const info = {
    1: { label: 'Excellent', color: '#43e97b', width: '20%' },
    2: { label: 'Bon', color: '#fddb92', width: '40%' },
    3: { label: 'Modéré', color: '#f6d365', width: '60%' },
    4: { label: 'Mauvais', color: '#ff758c', width: '80%' },
    5: { label: 'Très Mauvais', color: '#ff0844', width: '100%' }
  }
  return info[index] || info[1];
}

const currentAqi = computed(() => getAqiInfo(aqiIndex.value));
</script>

<template>
  <div class="widget-card aqi-widget">
    <div class="widget-header">
      <i class="ri-leaf-line"></i>
      <span>Qualité de l'air</span>
    </div>
    
    <div class="aqi-content">
      <div class="aqi-value" :style="{ color: currentAqi.color }">
        {{ currentAqi.label }}
      </div>
      
      <div class="progress-bar-container">
        <div class="progress-bar-fill" :style="{ width: currentAqi.width, background: currentAqi.color }"></div>
      </div>
      
      <div class="aqi-details" v-if="aqiData?.list?.[0]?.components">
        <div class="pollutant">
          <span>PM2.5</span>
          <strong>{{ aqiData.list[0].components.pm2_5 }}</strong>
        </div>
        <div class="pollutant">
          <span>CO</span>
          <strong>{{ aqiData.list[0].components.co }}</strong>
        </div>
        <div class="pollutant">
          <span>NO2</span>
          <strong>{{ aqiData.list[0].components.no2 }}</strong>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.widget-card {
  background: var(--widget-bg, rgba(255, 255, 255, 0.4));
  border-radius: 20px;
  padding: 20px;
  border: 1px solid rgba(255,255,255,0.2);
  display: flex;
  flex-direction: column;
  height: 100%;
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

.widget-header i {
  font-size: 1.1rem;
}

.aqi-content {
  flex: 1;
  display: flex;
  flex-direction: column;
  justify-content: center;
}

.aqi-value {
  font-size: 1.5rem;
  font-weight: 800;
  margin-bottom: 10px;
}

.progress-bar-container {
  height: 8px;
  background: rgba(0,0,0,0.1);
  border-radius: 10px;
  overflow: hidden;
  margin-bottom: 15px;
}

.progress-bar-fill {
  height: 100%;
  border-radius: 10px;
  transition: width 1s ease-in-out;
}

.aqi-details {
  display: flex;
  justify-content: space-between;
  border-top: 1px solid rgba(255,255,255,0.2);
  padding-top: 10px;
}

.pollutant {
  display: flex;
  flex-direction: column;
  align-items: center;
}

.pollutant span {
  font-size: 0.7rem;
  opacity: 0.7;
}

.pollutant strong {
  font-size: 0.9rem;
  font-weight: 700;
}
</style>
