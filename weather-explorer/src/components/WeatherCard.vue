<template>
  <div class="weather-card">
    <h2>🌤️ Previsão de hoje</h2>
    <div v-if="weather" class="weather-info">
      <div class="weather-main">
        <div class="weather-icon">
          {{ getWeatherIcon(weather.today.weathercode) }}
        </div>
        <div class="weather-temp">
          {{ weather.today.temp }}°C
        </div>
        <div class="weather-desc">
          {{ getWeatherDescription(weather.today.weathercode) }}
        </div>
      </div>

      <div class="weather-details">
        <div class="detail-item">
          <span class="detail-label"><strong>🌡️ Máx / Mín:</strong> {{ weather.today.max }}° / {{ weather.today.min }}°</span>
        </div>
        <div class="detail-item">
          <span class="detail-label"><strong>💨 Vento: </strong>{{ weather.today.wind }} km/h</span>
          <span class="detail-value"></span>
        </div>
        <div class="detail-item">
          <span class="detail-label"><strong>📅 Condição: </strong>{{ getWeatherDescription(weather.today.weathercode) }}</span>
          <span class="detail-value"></span>
        </div>
      </div>
    </div>
    <div v-else class="weather-info placeholder">
      <p>🔍 Use a busca acima para ver o clima</p>
    </div>
  </div>
</template>

<script setup>
defineProps({
  weather: Object
})

// Mesma função de ícone do HourlyCarousel (pode ser movida para um utilitário)
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

function getWeatherDescription(code) {
  const descriptions = {
    0: 'Céu limpo', 1: 'Principalmente limpo', 2: 'Parcialmente nublado', 3: 'Encoberto',
    45: 'Nevoeiro', 48: 'Nevoeiro com gelo',
    51: 'Garoa leve', 53: 'Garoa moderada', 55: 'Garoa densa',
    56: 'Garoa congelante', 57: 'Garoa congelante densa',
    61: 'Chuva leve', 63: 'Chuva moderada', 65: 'Chuva forte',
    66: 'Chuva congelante', 67: 'Chuva congelante forte',
    71: 'Neve leve', 73: 'Neve moderada', 75: 'Neve forte', 77: 'Grãos de neve',
    80: 'Pancadas leves', 81: 'Pancadas moderadas', 82: 'Pancadas fortes',
    85: 'Neve leve', 86: 'Neve forte',
    95: 'Trovoada', 96: 'Trovoada com granizo', 99: 'Trovoada forte'
  }
  return descriptions[code] || 'Condição variável'
}
</script>

<style scoped>
.weather-card h2 {
  font-size: 1.4rem;
  font-weight: 600;
  color: #1e2f3e;
  margin-bottom: 1rem;
  padding-left: 0.75rem;
}

.weather-info {
  background: #ffffff;
  border-radius: 1.25rem;
  padding: 1.5rem;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.04);
}

.weather-main {
  text-align: center;
  margin-bottom: 1.5rem;
}

.weather-icon {
  font-size: 3rem;
  margin-bottom: 0.5rem;
}

.weather-temp {
  font-size: 3rem;
  font-weight: 700;
  color: #1e4668;
  line-height: 1;
}

.weather-desc {
  font-size: 1rem;
  color: #4a627a;
  margin-top: 0.25rem;
  text-transform: capitalize;
}

.weather-details {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(140px, 1fr));
  gap: 1rem;
  border-top: 1px solid #e9ecef;
  padding-top: 1rem;
  justify-content: center;
  align-items: center;
}

.detail-item {
  text-align: center;
  font-size: 0.9rem;
}

.detail-label {
  color: #5d6f83;
  font-weight: 500;
}

.detail-value {
  font-weight: 600;
  color: #1e3c72;
}

.placeholder {
  text-align: center;
  color: #6c7a8a;
  background: #f8fafc;
}
</style>