<template>
  <div class="app-container">
    <header>
      <h1> <i class="fa-solid fa-cloud-sun"></i> Clima Bom</h1>
      <p>Previsão do tempo + recomendação de roupa</p>
    </header>

    <!-- Seção 1: Busca -->
    <section class="search-section">
      <SearchSection />
    </section>

    <!-- Seção 2: Card de previsão -->
    <section class="weather-section">
      <WeatherCard />
    </section>

    <!-- Seção 3: Card de recomendação + outfits -->
    <section class="recommendation-section">
      <RecommendationCard />
    </section>

    <footer>
      <small>Dados de previsão: Open-Meteo</small>
    </footer>
  </div>
</template>

<script setup>
// Por enquanto, vamos importar os componentes vazios (vamos criá-los depois)
import SearchSection from './components/SearchSection.vue'
import WeatherCard from './components/WeatherCard.vue'
import RecommendationCard from './components/RecommendationCard.vue'
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
.weather-section,
.recommendation-section {
  background: rgba(255, 255, 255, 0.85);
  backdrop-filter: blur(8px);
  border-radius: 2rem;
  padding: 1.5rem;
  box-shadow: 0 8px 20px rgba(0, 0, 0, 0.1);
  transition: transform 0.2s ease;
}

.search-section:hover,
.weather-section:hover,
.recommendation-section:hover {
  transform: translateY(-4px);
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
}
</style>