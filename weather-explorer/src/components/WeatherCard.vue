<template>
  <div class="weather-card">
    <h2>🌤️ Previsão de hoje</h2>
    <div v-if="weather" class="weather-info">
      <p><strong>Temperatura atual:</strong> {{ weather.today.temp }}°C</p>
      <p><strong>Máx / Mín:</strong> {{ weather.today.max }}° / {{ weather.today.min }}°</p>
      <p><strong>Vento:</strong> {{ weather.today.wind }} km/h</p>
      <p><strong>Código do clima:</strong> {{ weather.today.weathercode }}</p>
    </div>
    <div v-else class="weather-info">
      <p>🔍 Use a busca acima para ver o clima</p>
    </div>
  </div>
</template>

<script setup>
defineProps({
  weather: Object
})
</script>

<style scoped>
/* mesmo estilo de antes */
.weather-card h2 {
  margin-bottom: 1rem;
  color: #1e3c72;
}
.weather-info {
  background: #eef2f7;
  border-radius: 1.5rem;
  padding: 1.5rem;
  text-align: center;
}
</style>