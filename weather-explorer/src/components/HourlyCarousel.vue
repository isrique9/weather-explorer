<template>
  <div class="hourly-forecast" v-if="hasHourlyData">
    <h3> Previsão por hora</h3>
    <div class="carousel-container">
      <div class="carousel" ref="carouselRef">
        <div 
          v-for="(hour, index) in hourlyData.time" 
          :key="index"
          class="hour-card"
          :class="{ 'current-hour': index === 0 }"
        >
          <div class="hour-time">{{ formatHour(hour) }}</div>
          <div class="hour-icon">{{ getWeatherIcon(hourlyData.weathercode[index]) }}</div>
          <div class="hour-temp">{{ hourlyData.temperature_2m[index] }}°C</div>
          <div class="hour-desc">{{ getShortWeatherDesc(hourlyData.weathercode[index]) }}</div>
        </div>
      </div>
    </div>
    <div class="carousel-controls">
      <button @click="scrollLeft" class="scroll-btn"><i class="fa-solid fa-angle-left"></i> Anterior</button>
      <button @click="scrollRight" class="scroll-btn">Próximo <i class="fa-solid fa-angle-right"></i></button>
    </div>
  </div>
  <div v-else class="hourly-forecast">
    <p>⏳ Dados horários indisponíveis</p>
  </div>
</template>

<script setup>
import { ref, computed, watch } from 'vue'

const props = defineProps({
  weather: { type: Object, required: true }
})

const carouselRef = ref(null)
const hourlyData = ref({ time: [], temperature_2m: [], weathercode: [] })

watch(() => props.weather, (newWeather) => {
  if (newWeather && newWeather.hourly) {
    hourlyData.value = newWeather.hourly
  }
}, { immediate: true, deep: true })

const hasHourlyData = computed(() => hourlyData.value.time?.length > 0)

function formatHour(timeString) {
  if (!timeString) return '--:--'
  const date = new Date(timeString)
  return date.getHours().toString().padStart(2, '0') + ':' + date.getMinutes().toString().padStart(2, '0')
}

function getWeatherIcon(code) {
  const icons = {
    0: '☀️', 1: '🌤️', 2: '⛅', 3: '☁️',
    45: '🌫️', 48: '🌫️',
    51: '🌦️', 53: '🌦️', 55: '🌧️',
    56: '❄️🌧️', 57: '❄️🌧️',
    61: '🌧️', 63: '🌧️', 65: '🌧️',
    66: '❄️🌧️', 67: '❄️🌧️',
    71: '🌨️', 73: '🌨️', 75: '🌨️', 77: '🌨️',
    80: '🌧️', 81: '🌧️', 82: '🌧️',
    85: '🌨️', 86: '🌨️',
    95: '⛈️', 96: '⛈️', 99: '⛈️'
  }
  return icons[code] || '🌡️'
}

function getShortWeatherDesc(code) {
  const desc = {
    0: 'Limpo', 1: 'P. limpo', 2: 'Nuvens', 3: 'Nublado',
    45: 'Nevoeiro', 48: 'Nevoeiro',
    51: 'Garoa', 53: 'Garoa', 55: 'Chuvisco',
    61: 'Chuva', 63: 'Chuva', 65: 'Chuva forte',
    71: 'Neve', 73: 'Neve', 75: 'Neve forte',
    80: 'Pancada', 81: 'Pancada', 82: 'Tempestade',
    95: 'Trovoada', 96: 'Trovoada', 99: 'Trovoada'
  }
  return desc[code] || 'Variável'
}

function scrollLeft() {
  carouselRef.value?.scrollBy({ left: -200, behavior: 'smooth' })
}
function scrollRight() {
  carouselRef.value?.scrollBy({ left: 200, behavior: 'smooth' })
}
</script>

<style scoped>
.hourly-forecast h3 {
  font-size: 1.2rem;
  font-weight: 600;
  color: #1e2f3e;
  margin-bottom: 1rem;
  padding-left: 0.75rem;
}

.carousel {
  display: flex;
  gap: 0.8rem;
  overflow-x: auto;
  scroll-behavior: smooth;
  padding: 0.5rem 0 1rem;
  scrollbar-width: thin;
}
.carousel::-webkit-scrollbar {
  height: 6px;
}
.carousel::-webkit-scrollbar-track {
  background: #e2e8f0;
  border-radius: 10px;
}
.carousel::-webkit-scrollbar-thumb {
  background: #94a3b8;
  border-radius: 10px;
}

.hour-card {
  min-width: 85px;
  background: white;
  border-radius: 1rem;
  padding: 0.75rem 0.5rem;
  text-align: center;
  box-shadow: 0 2px 6px rgba(0, 0, 0, 0.04);
  transition: transform 0.1s ease;
  border: 1px solid #edf2f7;
  user-select: none;
}
.hour-card:hover {
  transform: translateY(-2px);
  border-color: #cbd5e1;
}

.hour-card.current-hour {
  background: #f8fafc;
  box-shadow: 0 4px 10px rgba(52, 152, 219, 0.1);
}

.hour-time {
  font-weight: 600;
  font-size: 0.8rem;
  color: #1e4668;
  margin-bottom: 0.5rem;
}
.hour-icon {
  font-size: 1.8rem;
  margin: 0.25rem 0;
}
.hour-temp {
  font-size: 1.2rem;
  font-weight: 700;
  color: #1e3c72;
}
.hour-desc {
  font-size: 0.7rem;
  color: #5d6f83;
  margin-top: 0.25rem;
  white-space: nowrap;
  overflow-x: hidden;
  text-overflow: ellipsis;
}

.carousel-controls {
  display: flex;
  justify-content: center;
  gap: 0.75rem;
  margin-top: 0.75rem;
}
.scroll-btn {
  background: #eef2f6;
  border: none;
  padding: 0.4rem 1rem;
  border-radius: 2rem;
  font-size: 0.8rem;
  font-weight: 500;
  color: #2c3e50;
  cursor: pointer;
  transition: background 0.2s;
}
.scroll-btn:hover {
  background: #e2e8f0;
}
@media (max-width: 640px) {
  .hour-card { min-width: 70px; }
  .hour-temp { font-size: 1rem; }
  .hour-icon { font-size: 1.4rem; }
}
</style>