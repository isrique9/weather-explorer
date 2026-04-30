<template>
  <div class="search-box">
    <h2>🌤️ Localização</h2>
    <button 
      class="location-btn" 
      @click="getUserLocation"
      :disabled="loading"
    >
      {{ loading ? 'Buscando...' : 'Usar minha localização' }}
    </button>
    <div class="manual-search">
      <input 
        type="text" 
        v-model="cityName"
        placeholder="Digite o nome da cidade..."
        @keyup.enter="searchByCity"
        @focus="selectInputText"
        @click="selectInputText"
        :disabled="loading"
      />
      <button 
        @click="searchByCity"
        :disabled="loading || !cityName.trim()"
      >
        Buscar
      </button>
    </div>
    <p v-if="errorMessage" class="error">{{ errorMessage }}</p>
  </div>
</template>

<script setup>
import { ref } from 'vue'

const emit = defineEmits(['weather-data'])
const cityName = ref('')
const loading = ref(false)
const errorMessage = ref('')

function selectInputText(event) {
  event.target.select()
}

// Função para buscar dados climáticos a partir de lat/lon
async function fetchWeather(lat, lon) {
  loading.value = true
  errorMessage.value = ''
  
  try {
    // URL corrigida - removido forecast_days=2 (padrão é 7)
    // e adicionado current_weather corretamente
    const url = `https://api.open-meteo.com/v1/forecast?latitude=${lat}&longitude=${lon}&current_weather=true&hourly=temperature_2m,weathercode&daily=temperature_2m_max,temperature_2m_min,weathercode,windspeed_10m_max&timezone=auto`

    
    const response = await fetch(url, {
      method: 'GET',
      mode: 'cors', // Explicitamente pedindo CORS
      headers: {
        'Accept': 'application/json',
      }
    })
    
    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`)
    }
    
    const data = await response.json()
    
    // Verifica se os dados horários existem
    if (!data.hourly || !data.hourly.time) {
      throw new Error('Dados de previsão por hora não disponíveis')
    }
    
    // Pega a hora atual do sistema
    const now = new Date()
    const currentHour = now.getHours()
    
    // Encontra o índice da primeira ocorrência com a mesma hora
    const startIndex = data.hourly.time.findIndex(time => {
      const hour = new Date(time).getHours()
      return hour === currentHour
    })
    
    const start = startIndex !== -1 ? startIndex : 0
    const hoursToShow = 24
    
    const hourlyData = {
      time: data.hourly.time.slice(start, start + hoursToShow),
      temperature_2m: data.hourly.temperature_2m.slice(start, start + hoursToShow),
      weathercode: data.hourly.weathercode.slice(start, start + hoursToShow)
    }
    
    if (hourlyData.time.length === 0) {
      hourlyData.time = data.hourly.time.slice(0, hoursToShow)
      hourlyData.temperature_2m = data.hourly.temperature_2m.slice(0, hoursToShow)
      hourlyData.weathercode = data.hourly.weathercode.slice(0, hoursToShow)
    }
    
    const weatherData = {
      today: {
        temp: data.current_weather.temperature,
        max: data.daily.temperature_2m_max[0],
        min: data.daily.temperature_2m_min[0],
        wind: data.current_weather.windspeed,
        weathercode: data.current_weather.weathercode
      },
      tomorrow: {
        max: data.daily.temperature_2m_max[1],
        min: data.daily.temperature_2m_min[1],
        wind: data.daily.windspeed_10m_max[1],
        weathercode: data.daily.weathercode[1]
      },
      hourly: hourlyData,
      location: { lat, lon }
    }
    
    emit('weather-data', weatherData)
  } catch (err) {
    console.error('Erro detalhado:', err)
    errorMessage.value = `Erro ao buscar clima: ${err.message}`
  } finally {
    loading.value = false
  }
}

// Resto do código permanece igual...
function getUserLocation() {
  if (!navigator.geolocation) {
    errorMessage.value = 'Seu navegador não suporta geolocalização.'
    return
  }
  
  loading.value = true
  errorMessage.value = ''
  
  navigator.geolocation.getCurrentPosition(
    (position) => {
      const { latitude, longitude } = position.coords
      fetchWeather(latitude, longitude)
    },
    (error) => {
      loading.value = false
      switch(error.code) {
        case error.PERMISSION_DENIED:
          errorMessage.value = 'Permissão negada. Use a busca manual.'
          break
        case error.POSITION_UNAVAILABLE:
          errorMessage.value = 'Localização indisponível.'
          break
        case error.TIMEOUT:
          errorMessage.value = 'Tempo limite excedido.'
          break
        default:
          errorMessage.value = 'Erro ao obter localização.'
      }
    }
  )
}

async function searchByCity() {
  if (!cityName.value.trim()) return
  
  loading.value = true
  errorMessage.value = ''
  
  try {
    const geoUrl = `https://geocoding-api.open-meteo.com/v1/search?name=${encodeURIComponent(cityName.value)}&count=1&language=pt&format=json`
    const response = await fetch(geoUrl)
    const data = await response.json()
    
    if (!data.results || data.results.length === 0) {
      throw new Error('Cidade não encontrada')
    }
    
    const { latitude, longitude, name, country } = data.results[0]
    console.log(`Cidade encontrada: ${name}, ${country}`)
    
    await fetchWeather(latitude, longitude)
  } catch (err) {
    errorMessage.value = err.message
    loading.value = false
  }
}
</script>

<style scoped>
.search-box {
  display: flex;
  flex-direction: column;
  gap: 1rem;
}
.location-btn {
  background: rgba(44, 62, 80, 0.85);
  backdrop-filter: blur(4px);
  color: white;
  border: none;
  padding: 0.75rem;
  border-radius: 8px;
  font-weight: bold;
  cursor: pointer;
  transition: background 0.2s;
}
.location-btn:hover:not(:disabled) {
  background: rgba(26, 42, 58, 0.9);
}
.location-btn:disabled {
  opacity: 0.6;
  cursor: not-allowed;
}
.manual-search {
  display: flex;
  gap: 0.5rem;
}
.manual-search input {
  flex: 1;
  padding: 0.75rem;
  border-radius: 8px;
  border: 1px solid rgba(255,255,255,0.5);
  background: rgba(255,255,255,0.7);
  font-size: 1rem;
  backdrop-filter: blur(4px);
}
.manual-search input:disabled {
  background-color: rgba(240, 240, 240, 0.5);
}
.manual-search button {
  background: rgba(52, 152, 219, 0.9);
  color: white;
  border: none;
  padding: 0 1.2rem;
  border-radius: 8px;
  cursor: pointer;
  transition: background 0.2s;
}
.manual-search button:hover:not(:disabled) {
  background: rgba(7, 124, 202, 0.95);
}
.manual-search button:disabled {
  opacity: 0.6;
  cursor: not-allowed;
}
.error {
  color: #b91c1c;
  font-size: 0.85rem;
  margin-top: 0.25rem;
  font-weight: 600;
  text-shadow: 0 0 2px white;
}
</style>