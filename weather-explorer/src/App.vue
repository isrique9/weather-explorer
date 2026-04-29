<template>
  <div class="app-container">
    <header>
      <h1>☀️ Clima Agora</h1>
      <p>Previsão do tempo para a sua cidade com poucos cliques.</p>
    </header>
    
    <section class="sun-section" v-if="showSun">
      <Sun />
    </section>

    <section class="search-section">
      <SearchSection @weather-data="handleWeatherData" />
    </section>

    <section class="weather-section">
      <WeatherCard :weather="weatherData" />
      <HourlyCarousel v-if="weatherData" :weather="weatherData" />
    </section>

    <footer>
      <small>Dados de previsão: Open-Meteo</small>
    </footer>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'
import SearchSection from './components/SearchSection.vue'
import WeatherCard from './components/WeatherCard.vue'
import HourlyCarousel from './components/HourlyCarousel.vue'
import Sun from './components/Sun.vue'

const weatherData = ref(null)

// Computed property para verificar se deve mostrar o sol
const showSun = computed(() => {
  return weatherData.value && weatherData.value.today.weathercode === 0
})

function handleWeatherData(data) {
  weatherData.value = data
}
</script>

<style scoped>
/* Reset básico e estilos globais */
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  font-family: system-ui, 'Segoe UI', Roboto, sans-serif;
  background: linear-gradient(145deg, #d4e6f1 0%, #f0f9ff 100%);
  min-height: 100vh;
}

.app-container {
  max-width: 1200px;
  margin: 0 auto;
  padding: 2rem 1.5rem;
  display: flex;
  flex-direction: column;
  gap: 2rem;
}

header {
  text-align: center;
  margin-bottom: 1rem;
}

header h1 {
  font-size: 2.2rem;
  background: linear-gradient(135deg, #1e3c72, #2b5297);
  background-clip: text;
  -webkit-background-clip: text;
  color: transparent;
  letter-spacing: -0.5px;
}

header p {
  color: #2c3e50;
  margin-top: 0.25rem;
  font-weight: 500;
}

/* Cada seção terá um card visual */
.search-section,
.weather-section {
  background: rgb(244, 249, 255);
  backdrop-filter: blur(8px);
  border-radius: 2rem;
  padding: 1.5rem;
}

/* Estilo para a seção do sol */
.sun-section {
  display: flex;
  justify-content: center;
  align-items: center;
  min-height: 300px;
}

footer {
  text-align: center;
  font-size: 0.75rem;
  color: #4a627a;
  margin-top: 1rem;
}

/* Responsividade */
@media (max-width: 768px) {
  .app-container {
    padding: 1rem;
    gap: 1.2rem;
  }
  header h1 {
    font-size: 1.6rem;
  }
  .sun-section {
    min-height: 200px;
  }
}
</style>