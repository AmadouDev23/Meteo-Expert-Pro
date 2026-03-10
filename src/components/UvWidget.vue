<script setup>
import { computed } from 'vue'

const props = defineProps({
  uvIndex: {
    type: Number,
    required: true
  }
})

const getUvInfo = (index) => {
  if (index < 3) return { label: 'Faible', color: '#43e97b', width: (index/11)*100 + '%' };
  if (index < 6) return { label: 'Modéré', color: '#f6d365', width: (index/11)*100 + '%' };
  if (index < 8) return { label: 'Élevé', color: '#ff9a44', width: (index/11)*100 + '%' };
  if (index < 11) return { label: 'Très Élevé', color: '#ff758c', width: (index/11)*100 + '%' };
  return { label: 'Extrême', color: '#ff0844', width: '100%' };
}

const currentUv = computed(() => getUvInfo(props.uvIndex));
</script>

<template>
  <div class="metric-card uv-widget">
    <div class="metric-header">
       <i class="ri-sun-cloudy-line"></i> Indice UV
    </div>
    
    <div class="uv-content">
      <div class="uv-value" :style="{ color: currentUv.color }">
        {{ Math.round(uvIndex) }}
      </div>
      <div class="uv-label">{{ currentUv.label }}</div>
      
      <div class="progress-bar-container">
        <div class="progress-bar-fill" :style="{ width: currentUv.width, background: currentUv.color }"></div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.metric-card {
  background: var(--widget-bg, rgba(255, 255, 255, 0.4));
  border-radius: 25px;
  padding: 20px;
  border: 1px solid rgba(255,255,255,0.2);
  display: flex;
  flex-direction: column;
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

.uv-content {
  flex: 1;
  display: flex;
  flex-direction: column;
  justify-content: center;
}

.uv-value {
  font-size: 2.5rem;
  font-weight: 800;
  line-height: 1;
  margin-bottom: 5px;
}

.uv-label {
  font-size: 1rem;
  font-weight: 600;
  margin-bottom: 15px;
}

.progress-bar-container {
  height: 8px;
  background: rgba(0,0,0,0.1);
  border-radius: 10px;
  overflow: hidden;
}

.progress-bar-fill {
  height: 100%;
  border-radius: 10px;
  transition: width 1s ease-in-out;
}
</style>
