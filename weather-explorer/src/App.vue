<template>
  <BackgroundEffect :weatherCode="weatherData?.today?.weathercode" />
  
  <div class="app-container">
    <header>
      <h1 :style="titleStyle">
        ☀️ Clima Agora
      </h1>
      <p :style="{ color: paragraphColor }">
        Previsão do tempo para a sua cidade com poucos cliques.
      </p>
    </header>
    
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
import BackgroundEffect from './components/BackgroundEffect.vue'

const weatherData = ref(null)

function handleWeatherData(data) {
  weatherData.value = data
}

// Gradiente dinâmico para o título baseado no código do tempo
const titleGradient = computed(() => {
  const code = weatherData.value?.today?.weathercode
  if (code === undefined || code === null) return 'linear-gradient(135deg, #1e3c72, #2b5297)'
  
  if (code === 0) return 'linear-gradient(135deg, #0f2b4d, #c2410c)'               
  if (code === 1 || code === 2 || code === 3) return 'linear-gradient(135deg, #0f172a, #334155)' 
  if (code === 45 || code === 48) return 'linear-gradient(135deg, #1e293b, #475569)'            
  if ((code >= 51 && code <= 65) || (code >= 80 && code <= 82)) return 'linear-gradient(135deg, #e0f2fe, #bae6fd)' 
  if ((code >= 71 && code <= 77) || (code >= 85 && code <= 86)) return 'linear-gradient(135deg, #0f172a, #1e3a8a)' 
  if (code >= 95 && code <= 99) return 'linear-gradient(135deg, #fde047, #f97316)'               
  
  return 'linear-gradient(135deg, #1e3c72, #2b5297)'
})

const paragraphColor = computed(() => {
  const code = weatherData.value?.today?.weathercode
  if (code === undefined || code === null) return '#1e293b'
  
  if (code === 0) return '#0f2b4d'          
  if (code === 1 || code === 2 || code === 3) return '#f1f5f9' 
  if (code === 45 || code === 48) return '#0f172a'              
  if ((code >= 51 && code <= 65) || (code >= 80 && code <= 82)) return '#f0f0f0' 
  if ((code >= 71 && code <= 77) || (code >= 85 && code <= 86)) return '#0f172a'  
  if (code >= 95 && code <= 99) return '#fef08a'                
  
  return '#1e293b'
})

const titleStyle = computed(() => ({
  backgroundImage: titleGradient.value,
  backgroundClip: 'text',
  WebkitBackgroundClip: 'text',
  color: 'transparent',
  userSelect: 'none',
  cursor: 'default'
}))
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
  min-height: 100vh;
}

.app-container {
  position: relative;
  max-width: 1200px;
  margin: 0 auto;
  padding: 2rem 1.5rem;
  display: flex;
  flex-direction: column;
  gap: 2rem;
  z-index: 1;
}

header {
  text-align: center;
  margin-bottom: 1rem;
}

header h1 {
  font-size: 2.2rem;
  letter-spacing: -0.5px;
  text-shadow: 0 2px 4px rgba(0, 0, 0, 0.2);
  display: inline-block;
}

header p {
  margin-top: 0.25rem;
  font-weight: 600;
  text-shadow: 0 1px 1px rgba(255,255,255,0.3);
  transition: color 0.3s ease;
}

/* Glassmorphism cards */
.search-section,
.weather-section {
  background: rgba(255, 255, 255, 0.35);
  backdrop-filter: blur(12px);
  border-radius: 2rem;
  padding: 1.5rem;
  border: 1px solid rgba(255, 255, 255, 0.4);
  box-shadow: 0 8px 20px rgba(0, 0, 0, 0.1);
}

footer {
  text-align: center;
  font-size: 0.75rem;
  color: #1e293b;
  margin-top: 1rem;
  font-weight: 500;
  text-shadow: 0 1px 1px rgba(255,255,255,0.3);
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
}
</style>