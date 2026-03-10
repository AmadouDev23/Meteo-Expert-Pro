<script setup>
import { computed } from 'vue'
import { Line } from 'vue-chartjs'
import {
  Chart as ChartJS,
  CategoryScale,
  LinearScale,
  PointElement,
  LineElement,
  Title,
  Tooltip,
  Filler
} from 'chart.js'

ChartJS.register(
  CategoryScale,
  LinearScale,
  PointElement,
  LineElement,
  Title,
  Tooltip,
  Filler
)

const props = defineProps({
  forecast: {
    type: Array,
    required: true
  }
})

// Next 24 hours (8 periods of 3 hours)
const hourlyData = computed(() => props.forecast.slice(0, 8))

const chartData = computed(() => {
  const labels = hourlyData.value.map((hour, index) => {
    if (index === 0) return 'Maint.';
    const date = new Date(hour.dt * 1000);
    return date.toLocaleTimeString('fr-FR', { hour: '2-digit', minute: '2-digit' }).replace(':', 'h');
  });

  const dataPoints = hourlyData.value.map(hour => Math.round(hour.main.temp));

  return {
    labels,
    datasets: [
      {
        label: 'Température (°)',
        data: dataPoints,
        borderColor: '#4facfe',
        backgroundColor: 'rgba(79, 172, 254, 0.2)',
        borderWidth: 3,
        pointBackgroundColor: '#fff',
        pointBorderColor: '#4facfe',
        pointRadius: 4,
        pointHoverRadius: 6,
        fill: true,
        tension: 0.4 // Smooth curve
      }
    ]
  }
})

const chartOptions = {
  responsive: true,
  maintainAspectRatio: false,
  plugins: {
    legend: { display: false },
    tooltip: {
      backgroundColor: 'rgba(0, 0, 0, 0.7)',
      titleFont: { size: 13 },
      bodyFont: { size: 14, weight: 'bold' },
      padding: 10,
      displayColors: false,
      callbacks: {
        label: (context) => `${context.parsed.y}°`
      }
    }
  },
  scales: {
    x: {
      grid: { display: false, drawBorder: false },
      ticks: { color: 'rgba(255, 255, 255, 0.8)', font: { size: 11 } }
    },
    y: {
      display: false, // Hide Y axis fully for a cleaner look
      grid: { display: false },
      min: Math.min(...hourlyData.value.map(h => h.main.temp)) - 5, // Give some bottom breathing room
      max: Math.max(...hourlyData.value.map(h => h.main.temp)) + 5
    }
  },
  animation: {
    duration: 1000,
    easing: 'easeOutQuart'
  }
}
</script>

<template>
  <div class="chart-widget">
    <div class="widget-header">
      <i class="ri-funds-line"></i>
      <span>Température (24h)</span>
    </div>
    
    <div class="chart-container">
      <Line :data="chartData" :options="chartOptions" />
    </div>
  </div>
</template>

<style scoped>
.chart-widget {
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

.chart-container {
  height: 180px;
  width: 100%;
  position: relative;
}

/* Local context styles for dark modes */
.body.rainy .chart-widget, .body.cloudy .chart-widget {
  background: rgba(0,0,0,0.3);
}
</style>
