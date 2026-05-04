<template>
  <div class="map-section" v-if="hasValidCoords">
    <h3>Localização no Mapa</h3>
    <div ref="mapContainer" class="map-container"></div>
  </div>
</template>

<script setup>
import { ref, watch, onUnmounted } from 'vue'
import L from 'leaflet'
import 'leaflet/dist/leaflet.css'

// Configuração do ícone padrão do Leaflet (para evitar problemas)
delete L.Icon.Default.prototype._getIconUrl
L.Icon.Default.mergeOptions({
  iconRetinaUrl: new URL('leaflet/dist/images/marker-icon-2x.png', import.meta.url).href,
  iconUrl: new URL('leaflet/dist/images/marker-icon.png', import.meta.url).href,
  shadowUrl: new URL('leaflet/dist/images/marker-shadow.png', import.meta.url).href,
})

const props = defineProps({
  location: {
    type: Object,
    default: null,
  },
})

const mapContainer = ref(null)
let mapInstance = null
let markerInstance = null
let boundaryLayer = null

const hasValidCoords = ref(false)

// Função para buscar o boundary (polígono) da cidade via Nominatim
async function fetchCityBoundary(lat, lon, cityName) {
  const url = `https://nominatim.openstreetmap.org/search?q=${encodeURIComponent(cityName)}&format=json&polygon_geojson=1&limit=1`
  
  try {
    const response = await fetch(url)
    const data = await response.json()
    
    if (data && data.length > 0 && data[0].geojson) {
      const geojson = data[0].geojson
      
      // Remove a camada anterior, se existir
      if (boundaryLayer) {
        mapInstance.removeLayer(boundaryLayer)
      }
      
      // Adiciona o polígono ao mapa com o estilo desejado
      boundaryLayer = L.geoJSON(geojson, {
        style: {
          color: '#037ee2',     
          weight: 3,            
          fillColor: '#279cfc', 
          fillOpacity: 0.1,     
          dashArray: '5, 10'    
        }
      }).addTo(mapInstance)
      
      // Ajusta o zoom para mostrar todo o polígono
      mapInstance.fitBounds(boundaryLayer.getBounds())
    } else {
      console.warn('Nenhum boundary encontrado para esta localização.')
    }
  } catch (error) {
    console.error('Erro ao buscar boundary da cidade:', error)
  }
}

// Função para inicializar o mapa (já existente, mas com pequenas adaptações)
function initMap(lat, lon, name) {
  if (!mapContainer.value) return
  
  if (mapInstance) {
    mapInstance.remove()
    mapInstance = null
  }
  
  // Cria o mapa
  mapInstance = L.map(mapContainer.value).setView([lat, lon], 12)
  
  L.tileLayer('https://{s}.basemaps.cartocdn.com/light_all/{z}/{x}/{y}{r}.png', {
    attribution: '&copy; <a href="https://www.openstreetmap.org/copyright">OSM</a> &copy; CartoDB',
    subdomains: 'abcd',
    maxZoom: 19,
  }).addTo(mapInstance)
  
  // Adiciona o marcador padrão
  markerInstance = L.marker([lat, lon])
    .addTo(mapInstance)
    .bindPopup(`<b>${name || 'Localização'}</b>`)
    .openPopup()
  
  // Busca e desenha o boundary da cidade
  fetchCityBoundary(lat, lon, name)
}

// Watcher para observar mudanças na localização
watch(
  () => props.location,
  (newLocation) => {
    if (newLocation && typeof newLocation.lat === 'number' && typeof newLocation.lon === 'number') {
      hasValidCoords.value = true
      setTimeout(() => {
        initMap(newLocation.lat, newLocation.lon, newLocation.name)
      }, 100)
    } else {
      hasValidCoords.value = false
      if (mapInstance) {
        mapInstance.remove()
        mapInstance = null
      }
    }
  },
  { immediate: true, deep: true }
)

onUnmounted(() => {
  if (mapInstance) {
    mapInstance.remove()
    mapInstance = null
  }
})
</script>

<style scoped>
/* Seus estilos existentes */
.map-section {
  margin-top: 2rem;
  background: rgba(255, 255, 255, 0.35);
  backdrop-filter: blur(12px);
  border-radius: 2rem;
  padding: 1.2rem;
}
.map-section h3 {
  margin: 0 0 0.8rem 0;
  color: #0f172a;
  padding-left: 0.5rem;
}
.map-container {
  width: 100%;
  height: 350px;
  border-radius: 1.2rem;
  overflow: hidden;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
}
.map-credit {
  text-align: center;
  font-size: 0.7rem;
  margin-top: 0.5rem;
  color: #1e293b;
  font-weight: 500;
}
@media (max-width: 640px) {
  .map-container {
    height: 250px;
  }
}
</style>