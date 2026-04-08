<template>
  <div class="hourly-forecast" v-if="hasHourlyData">
    <h3>📊 Previsão por hora</h3>
    <div class="carousel-container">
      <div class="carousel" ref="carouselRef">
        <div 
          v-for="(hour, index) in hourlyData.time" 
          :key="index"
          class="hour-card"
          :class="{ 'current-hour': index === 0 }"
        >
          <div class="hour-time">{{ formatHour(hour) }}</div>
          <div class="hour-temp">{{ hourlyData.temperature_2m[index] }}°C</div>
          <div class="hour-icon">{{ getWeatherIcon(hourlyData.weathercode[index]) }}</div>
        </div>
      </div>
    </div>
    <div class="carousel-controls">
      <button @click="scrollLeft" class="scroll-btn">◀</button>
      <button @click="scrollRight" class="scroll-btn">▶</button>
    </div>
  </div>
  <div v-else class="hourly-forecast">
    <p>⏳ Dados horários indisponíveis</p>
  </div>
</template>

<script setup>
import { ref, computed, watch } from 'vue'

const props = defineProps({
  weather: {
    type: Object,
    required: true
  }
})

const carouselRef = ref(null)
const hourlyData = ref({ time: [], temperature_2m: [], weathercode: [] })

// Atualiza os dados sempre que a prop weather mudar
watch(() => props.weather, (newWeather) => {
  if (newWeather && newWeather.hourly) {
    hourlyData.value = newWeather.hourly
    console.log('Carrossel atualizado:', hourlyData.value)
  }
}, { immediate: true, deep: true })

const hasHourlyData = computed(() => {
  return hourlyData.value.time && hourlyData.value.time.length > 0
})

function formatHour(timeString) {
  if (!timeString) return '--:--'
  const date = new Date(timeString)
  const hours = date.getHours().toString().padStart(2, '0')
  const minutes = date.getMinutes().toString().padStart(2, '0')
  return `${hours}:${minutes}`
}

function getWeatherIcon(code) {
  const icons = {
    0: '☀️',   // Céu limpo
    1: '🌤️',   // Principalmente limpo
    2: '⛅',    // Parcialmente nublado
    3: '☁️',    // Encoberto
    45: '🌫️',  // Nevoeiro
    48: '🌫️',  // Nevoeiro com gelo
    51: '🌦️',  // Garoa leve
    53: '🌦️',  // Garoa moderada
    55: '🌧️',  // Garoa densa
    56: '❄️🌧️',// Garoa congelante leve
    57: '❄️🌧️',// Garoa congelante densa
    61: '🌧️',  // Chuva leve
    63: '🌧️',  // Chuva moderada
    65: '🌧️',  // Chuva forte
    66: '❄️🌧️',// Chuva congelante leve
    67: '❄️🌧️',// Chuva congelante forte
    71: '🌨️',  // Neve leve
    73: '🌨️',  // Neve moderada
    75: '🌨️',  // Neve forte
    77: '🌨️',  // Grãos de neve
    80: '🌧️',  // Pancadas de chuva leves
    81: '🌧️',  // Pancadas moderadas
    82: '🌧️',  // Pancadas fortes
    85: '🌨️',  // Pancadas de neve leves
    86: '🌨️',  // Pancadas de neve fortes
    95: '⛈️',   // Trovoada
    96: '⛈️',   // Trovoada com granizo leve
    99: '⛈️'    // Trovoada com granizo forte
  }
  return icons[code] || '🌡️'
}

function scrollLeft() {
  if (carouselRef.value) {
    carouselRef.value.scrollBy({ left: -200, behavior: 'smooth' })
  }
}

function scrollRight() {
  if (carouselRef.value) {
    carouselRef.value.scrollBy({ left: 200, behavior: 'smooth' })
  }
}
</script>

<style scoped>
.hourly-forecast {
  margin-top: 1rem;
}

.hourly-forecast h3 {
  color: #1e3c72;
  margin-bottom: 0.75rem;
}

.carousel-container {
  position: relative;
  overflow-x: auto;
  scroll-behavior: smooth;
  -webkit-overflow-scrolling: touch;
}

.carousel {
  display: flex;
  gap: 1rem;
  overflow-x: auto;
  scroll-behavior: smooth;
  padding: 0.5rem 0;
}

.carousel::-webkit-scrollbar {
  display: none;
}

.hour-card {
  min-width: 80px;
  background: rgba(255, 255, 255, 0.9);
  border-radius: 1rem;
  padding: 0.75rem;
  text-align: center;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
  transition: transform 0.2s;
}

.hour-card:hover {
  transform: translateY(-4px);
}

.hour-card.current-hour {
  background: #3498db;
  color: white;
  box-shadow: 0 4px 12px rgba(52, 152, 219, 0.3);
}

.hour-time {
  font-weight: bold;
  font-size: 0.9rem;
  margin-bottom: 0.5rem;
}

.hour-temp {
  font-size: 1.2rem;
  font-weight: bold;
  margin-bottom: 0.5rem;
}

.hour-icon {
  font-size: 1.5rem;
}

.carousel-controls {
  display: flex;
  justify-content: center;
  gap: 0.5rem;
  margin-top: 1rem;
}

.scroll-btn {
  background: #2c3e50;
  color: white;
  border: none;
  padding: 0.5rem 1rem;
  border-radius: 2rem;
  cursor: pointer;
  transition: background 0.2s;
}

.scroll-btn:hover {
  background: #1a2a3a;
}

@media (max-width: 768px) {
  .hour-card {
    min-width: 70px;
    padding: 0.5rem;
  }
  .hour-temp {
    font-size: 1rem;
  }
  .hour-icon {
    font-size: 1.2rem;
  }
}
</style>