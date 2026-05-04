<template>
  <div class="search-box">
    <h2>🌤️ Buscar Localização</h2>
    <button 
      class="location-btn" 
      @click="getUserLocation"
      :disabled="loading"
    >
      {{ loading ? 'Buscando...' : 'Usar minha localização' }}
    </button>

    <div class="manual-search">
      <div class="search-fields">
        <input 
          type="text" 
          v-model="cityName"
          placeholder="Cidade *"
          @keyup.enter="searchByCity"
          :disabled="loading"
          @focus="handleInputFocus"
        />
        <input 
          type="text" 
          v-model="stateName"
          placeholder="Estado (ex: SP)"
          @keyup.enter="searchByCity"
          :disabled="loading"
          @focus="handleInputFocus"
        />
        <input 
          type="text" 
          v-model="countryName"
          placeholder="País (ex: Brasil)"
          @keyup.enter="searchByCity"
          :disabled="loading"
          @focus="handleInputFocus"
        />
      </div>
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
const stateName = ref('')
const countryName = ref('')
const loading = ref(false)
const errorMessage = ref('')

// Mapeamento de siglas para nomes completos de estados do Brasil
const estadoMap = {
  'ac': 'acre', 'al': 'alagoas', 'ap': 'amapá', 'am': 'amazonas',
  'ba': 'bahia', 'ce': 'ceará', 'df': 'distrito federal', 'es': 'espírito santo',
  'go': 'goiás', 'ma': 'maranhão', 'mt': 'mato grosso', 'ms': 'mato grosso do sul',
  'mg': 'minas gerais', 'pa': 'pará', 'pb': 'paraíba', 'pr': 'paraná',
  'pe': 'pernambuco', 'pi': 'piauí', 'rj': 'rio de janeiro', 'rn': 'rio grande do norte',
  'rs': 'rio grande do sul', 'ro': 'rondônia', 'rr': 'roraima', 'sc': 'santa catarina',
  'sp': 'são paulo', 'se': 'sergipe', 'to': 'tocantins'
}

// Seleciona todo o texto do input ao focar
function handleInputFocus(event) {
  event.target.select()
}

// Função para buscar dados climáticos a partir de lat/lon e um nome opcional
async function fetchWeather(lat, lon, locationName = null) {
  loading.value = true
  errorMessage.value = ''
  
  try {
    const url = `https://api.open-meteo.com/v1/forecast?latitude=${lat}&longitude=${lon}&current_weather=true&hourly=temperature_2m,weathercode&daily=temperature_2m_max,temperature_2m_min,weathercode,windspeed_10m_max&timezone=auto`

    const response = await fetch(url, {
      method: 'GET',
      mode: 'cors',
      headers: { 'Accept': 'application/json' }
    })
    
    if (!response.ok) throw new Error(`HTTP error! status: ${response.status}`)
    
    const data = await response.json()
    
    if (!data.hourly || !data.hourly.time) {
      throw new Error('Dados de previsão por hora não disponíveis')
    }
    
    const now = new Date()
    const currentHour = now.getHours()
    const startIndex = data.hourly.time.findIndex(time => new Date(time).getHours() === currentHour)
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
      daily: {
        time: data.daily.time,
        temperature_2m_max: data.daily.temperature_2m_max,
        temperature_2m_min: data.daily.temperature_2m_min,
        weathercode: data.daily.weathercode,
        windspeed_10m_max: data.daily.windspeed_10m_max,
        precipitation_probability_max: data.daily.precipitation_probability_max,
        uv_index_max: data.daily.uv_index_max
      },
      hourly: hourlyData,
      location: {
        lat,
        lon,
        name: locationName || `Coordenadas: ${lat.toFixed(2)}, ${lon.toFixed(2)}`
      }
    }
    
    emit('weather-data', weatherData)
  } catch (err) {
    console.error('Erro detalhado:', err)
    errorMessage.value = `Erro ao buscar clima: ${err.message}`
  } finally {
    loading.value = false
  }
}

// Função para geolocalização
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

// Busca por cidade + estado + país
async function searchByCity() {
  const cidade = cityName.value.trim()
  if (!cidade) return

  loading.value = true
  errorMessage.value = ''

  try {
    // Busca apenas pela cidade (retorna até 10 resultados)
    const geoUrl = `https://geocoding-api.open-meteo.com/v1/search?name=${encodeURIComponent(cidade)}&count=10&language=pt&format=json`
    const response = await fetch(geoUrl)
    const data = await response.json()
    
    if (!data.results || data.results.length === 0) {
      throw new Error(`Nenhum local encontrado para "${cidade}"`)
    }

    let resultados = data.results
    let estadoDigitado = stateName.value.trim().toLowerCase()
    const paisDigitado = countryName.value.trim().toLowerCase()

    // Se o estado digitado for uma sigla, converte para nome completo
    if (estadoDigitado && estadoMap[estadoDigitado]) {
      estadoDigitado = estadoMap[estadoDigitado]
    }

    // Filtra por estado e país
    if (estadoDigitado || paisDigitado) {
      resultados = resultados.filter(local => {
        let match = true
        if (estadoDigitado) {
          const estadoLocalRaw = (local.admin1 || '').toLowerCase()
          // Tenta normalizar o estado local também (caso ele venha como sigla)
          let estadoLocalNormalizado = estadoLocalRaw
          for (const [sigla, nome] of Object.entries(estadoMap)) {
            if (estadoLocalRaw === sigla) {
              estadoLocalNormalizado = nome
              break
            }
          }
          // Verifica se o estado digitado está contido no nome original ou normalizado
          match = match && (estadoLocalRaw.includes(estadoDigitado) || estadoLocalNormalizado.includes(estadoDigitado))
        }
        if (paisDigitado) {
          const paisLocal = (local.country || '').toLowerCase()
          match = match && paisLocal.includes(paisDigitado)
        }
        return match
      })
    }

    if (resultados.length === 0) {
      let sugestao = `Nenhum local encontrado para "${cidade}"`
      if (estadoDigitado) sugestao += ` com estado "${stateName.value.trim()}"`
      if (paisDigitado) sugestao += ` com país "${countryName.value.trim()}"`
      sugestao += `. Tente buscar apenas a cidade ou verifique os nomes.`
      throw new Error(sugestao)
    }

    const resultado = resultados[0]
    const { latitude, longitude, name, admin1, country } = resultado

    const displayParts = [name]
    if (admin1) displayParts.push(admin1)
    if (country) displayParts.push(country)
    const locationName = displayParts.join(', ')

    await fetchWeather(latitude, longitude, locationName)
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
  flex-direction: column;
  gap: 0.75rem;
}
.search-fields {
  display: flex;
  flex-wrap: wrap;
  gap: 0.5rem;
}
.search-fields input {
  flex: 1;
  min-width: 120px;
  padding: 0.75rem;
  border-radius: 8px;
  border: 1px solid rgba(255,255,255,0.5);
  background: rgba(255,255,255,0.7);
  font-size: 1rem;
  backdrop-filter: blur(4px);
}
.search-fields input:disabled {
  background-color: rgba(240, 240, 240, 0.5);
}
.manual-search button {
  background: rgba(52, 152, 219, 0.9);
  color: white;
  border: none;
  padding: 0.75rem;
  border-radius: 8px;
  cursor: pointer;
  transition: background 0.2s;
  font-weight: bold;
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