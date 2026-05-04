<!-- WeeklyChart.vue -->
<template>
  <div class="weekly-chart" v-if="dailyData && dailyData.time">
    <div class="chart-header">
      <h3>📊 Previsão para 7 dias</h3>
      <div class="unit-toggle">
        <button @click="toggleUnit" :class="{ active: unit === 'c' }">°C</button>
        <button @click="toggleUnit" :class="{ active: unit === 'f' }">°F</button>
      </div>
    </div>

    <div class="chart-container" ref="chartContainer">
      <canvas ref="chartCanvas"></canvas>
    </div>

    <div class="chart-legend">
      <span><span class="color-min"></span> Temp. mínima</span>
      <span><span class="color-max"></span> Temp. máxima</span>
      <span><span class="color-precip"></span> Prob. chuva (%)</span>
      <span><span class="color-wind"></span> Vento máx. (km/h)</span>
    </div>
  </div>
</template>

<script setup>
import { ref, watch, onMounted, onUnmounted, computed } from 'vue'
import { Chart, registerables } from 'chart.js'
import ChartDataLabels from 'chartjs-plugin-datalabels'
Chart.register(...registerables, ChartDataLabels)

const props = defineProps({
  weather: Object   // objeto completo retornado da API
})

const emit = defineEmits(['day-selected'])

const dailyData = computed(() => props.weather?.daily || null)
const unit = ref('c') // 'c' ou 'f'
const chartCanvas = ref(null)
const chartContainer = ref(null)
let chartInstance = null

function toggleUnit() {
  unit.value = unit.value === 'c' ? 'f' : 'c'
  if (chartInstance) {
    updateChart()
  }
}

function convertTemp(celsius) {
  if (unit.value === 'f') return (celsius * 9/5 + 32).toFixed(0)
  return celsius.toFixed(0)
}

function getWeatherIcon(code) {
  const icons = {
    0: '☀️', 1: '🌤️', 2: '⛅', 3: '☁️',
    45: '🌫️', 48: '🌫️',
    51: '🌦️', 53: '🌦️', 55: '🌧️',
    61: '🌧️', 63: '🌧️', 65: '🌧️',
    71: '🌨️', 73: '🌨️', 75: '🌨️', 77: '🌨️',
    80: '🌧️', 81: '🌧️', 82: '🌧️',
    85: '🌨️', 86: '🌨️',
    95: '⛈️', 96: '⛈️', 99: '⛈️'
  }
  return icons[code] || '🌡️'
}

function formatDayLabel(dateStr) {
  const date = new Date(dateStr)
  return date.toLocaleDateString('pt-BR', { weekday: 'short' })
}

function updateChart() {
  if (!chartCanvas.value || !dailyData.value) return
  const ctx = chartCanvas.value.getContext('2d')
  const labels = dailyData.value.time.map(t => formatDayLabel(t))
  const maxTemps = dailyData.value.temperature_2m_max.map(v => parseFloat(convertTemp(v)))
  const minTemps = dailyData.value.temperature_2m_min.map(v => parseFloat(convertTemp(v)))
  const precipProb = dailyData.value.precipitation_probability_max || []
  const windSpeed = dailyData.value.windspeed_10m_max || []  // km/h
  const weatherCodes = dailyData.value.weathercode

  if (chartInstance) chartInstance.destroy()

  chartInstance = new Chart(ctx, {
    type: 'line',
    data: {
      labels: labels,
      datasets: [
        {
          label: 'Máxima (°' + (unit.value === 'c' ? 'C' : 'F') + ')',
          data: maxTemps,
          borderColor: '#e63946',
          backgroundColor: 'rgba(230, 57, 70, 0.1)',
          tension: 0.3,
          pointRadius: 6,
          pointHoverRadius: 8,
          pointBackgroundColor: '#e63946',
          yAxisID: 'y',
          fill: false
        },
        {
          label: 'Mínima (°' + (unit.value === 'c' ? 'C' : 'F') + ')',
          data: minTemps,
          borderColor: '#1e88e5',
          backgroundColor: 'rgba(30, 136, 229, 0.1)',
          tension: 0.3,
          pointRadius: 6,
          pointHoverRadius: 8,
          pointBackgroundColor: '#1e88e5',
          yAxisID: 'y',
          fill: false
        },
        {
          label: 'Probabilidade de chuva (%)',
          data: precipProb,
          type: 'bar',
          backgroundColor: 'rgba(54, 162, 235, 0.4)',
          yAxisID: 'y1',
          barPercentage: 0.6,
          categoryPercentage: 0.8,
          borderRadius: 4
        },
        {
          label: 'Vento máximo (km/h)',
          data: windSpeed,
          type: 'line',
          borderColor: '#2ecc71',
          backgroundColor: 'transparent',
          borderDash: [5, 5],
          tension: 0.3,
          pointRadius: 4,
          pointBackgroundColor: '#2ecc71',
          yAxisID: 'y1',
          fill: false
        }
      ]
    },
    options: {
      responsive: true,
      maintainAspectRatio: false,
      interaction: {
        mode: 'index',
        intersect: false,
      },
      plugins: {
        tooltip: {
          callbacks: {
            label: (context) => {
              let label = context.dataset.label || ''
              let val = context.raw
              if (context.dataset.label.includes('Máxima') || context.dataset.label.includes('Mínima')) {
                label += `: ${val}°${unit.value === 'c' ? 'C' : 'F'}`
              } else if (context.dataset.label.includes('Vento')) {
                label += `: ${val} km/h`
              } else {
                label += `: ${val}%`
              }
              // Informações extras do dia (ventos, UV)
              const idx = context.dataIndex
              const wind = dailyData.value.windspeed_10m_max?.[idx] || '--'
              const uv = dailyData.value.uv_index_max?.[idx] || '--'
              label += `\n💨 Vento: ${wind} km/h`
              label += `\n☀️ UV: ${uv}`
              return label
            }
          }
        },
        datalabels: {
          align: 'top',
          offset: 8,
          formatter: (value, context) => {
            // Mostra ícone apenas na série de máxima (primeira dataset)
            if (context.datasetIndex === 0) {
              const idx = context.dataIndex
              const icon = getWeatherIcon(weatherCodes[idx])
              return icon
            }
            return null
          },
          font: { size: 14 }
        },
        legend: { position: 'top' }
      },
      scales: {
        y: {
          title: { display: true, text: `Temperatura (°${unit.value === 'c' ? 'C' : 'F'})` },
          min: Math.min(...minTemps) - 2,
          max: Math.max(...maxTemps) + 2
        },
        y1: {
          position: 'right',
          title: { display: true, text: 'Chuva (%) / Vento (km/h)' },
          min: 0,
          max: Math.max(100, ...windSpeed) + 5,
          grid: { drawOnChartArea: false }
        }
      },
      onClick: (event, activeElements) => {
        if (activeElements.length > 0) {
          const index = activeElements[0].index
          const selectedDate = dailyData.value.time[index]
          emit('day-selected', selectedDate)  // envia apenas a data ISO
        }
      }
    },
    plugins: [ChartDataLabels]
  })

  // destaque do dia atual (primeiro dia)
  if (chartInstance && dailyData.value.time.length > 0) {
    const todayIndex = 0
    chartInstance.data.datasets.forEach(dataset => {
      if (dataset.pointBackgroundColor && !Array.isArray(dataset.pointBackgroundColor)) {
        const colors = new Array(dataset.data.length).fill(dataset.pointBackgroundColor)
        colors[todayIndex] = '#ffb74d'
        dataset.pointBackgroundColor = colors
        const borders = new Array(dataset.data.length).fill(dataset.pointBorderColor || dataset.borderColor)
        borders[todayIndex] = '#ff9800'
        dataset.pointBorderColor = borders
      } else if (Array.isArray(dataset.pointBackgroundColor)) {
        dataset.pointBackgroundColor[todayIndex] = '#ffb74d'
        if (dataset.pointBorderColor) dataset.pointBorderColor[todayIndex] = '#ff9800'
      }
    })
    chartInstance.update()
  }
}

watch(() => props.weather, () => {
  if (props.weather && props.weather.daily) {
    updateChart()
  }
}, { immediate: true, deep: true, flush: 'post' })

watch(unit, () => updateChart())

onMounted(() => {
  if (props.weather && props.weather.daily) updateChart()
})

onUnmounted(() => {
  if (chartInstance) chartInstance.destroy()
})
</script>

<style scoped>
.weekly-chart {
  margin-top: 2rem;
  background: rgba(255,255,255,0.35);
  backdrop-filter: blur(12px);
  border-radius: 2rem;
  padding: 1.2rem;
}
.chart-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  flex-wrap: wrap;
  margin-bottom: 1rem;
}
.chart-header h3 {
  margin: 0;
  color: #0f172a;
}
.unit-toggle button {
  background: #eef2f6;
  border: none;
  padding: 0.3rem 0.8rem;
  border-radius: 1rem;
  margin-left: 0.3rem;
  cursor: pointer;
  font-weight: bold;
}
.unit-toggle button.active {
  background: #3498db;
  color: white;
}
.chart-container {
  width: 100%;
  overflow-x: auto;
  position: relative;
}
canvas {
  max-height: 400px;
  width: 100%;
  min-width: 600px;
}
.chart-legend {
  display: flex;
  justify-content: center;
  gap: 1.5rem;
  margin-top: 1rem;
  font-size: 0.8rem;
  flex-wrap: wrap;
}
.color-min, .color-max, .color-precip, .color-wind {
  display: inline-block;
  width: 20px;
  height: 10px;
  border-radius: 4px;
  margin-right: 6px;
}
.color-min { background-color: #1e88e5; }
.color-max { background-color: #e63946; }
.color-precip { background-color: #36a2eb; }
.color-wind { background-color: #2ecc71; }

/* Esconde o gráfico em telas menores que 768px (mobile) */
@media (max-width: 767px) {
  .weekly-chart {
    display: none;
  }
}
</style>