<!-- components/BackgroundEffect.vue -->
<template>
  <div class="background-effect" :class="bgClass">
    <canvas ref="canvasRef" v-if="showParticleEffect" class="particle-canvas"></canvas>
    <div v-if="isThunderstorm" class="lightning-flash" :class="{ active: lightningActive }"></div>
  </div>
</template>

<script setup>
import { ref, watch, onMounted, onUnmounted, computed } from 'vue'

const props = defineProps({
  weatherCode: Number
})

const canvasRef = ref(null)
let animationId = null
let particles = []
let ctx = null
let canvasWidth = 0, canvasHeight = 0
const lightningActive = ref(false)
let lightningInterval = null

// Determine background class and particle effect
const bgClass = computed(() => {
  const code = props.weatherCode
  if (code === undefined || code === null) return 'bg-default'
  if (code === 0) return 'bg-clear'
  if (code === 1 || code === 2) return 'bg-partly-cloudy'
  if (code === 3) return 'bg-overcast'
  if (code === 45 || code === 48) return 'bg-fog'
  if ((code >= 51 && code <= 65) || (code >= 80 && code <= 82)) return 'bg-rain'
  if ((code >= 71 && code <= 77) || (code >= 85 && code <= 86)) return 'bg-snow'
  if (code >= 95 && code <= 99) return 'bg-thunderstorm'
  return 'bg-default'
})

const showParticleEffect = computed(() => {
  const code = props.weatherCode
  if ((code >= 51 && code <= 65) || (code >= 80 && code <= 82)) return true
  if ((code >= 71 && code <= 77) || (code >= 85 && code <= 86)) return true
  return false
})

const isThunderstorm = computed(() => {
  const code = props.weatherCode
  return code >= 95 && code <= 99
})

function initParticles(type) {
  particles = []
  if (!canvasRef.value) return
  canvasWidth = window.innerWidth
  canvasHeight = window.innerHeight
  canvasRef.value.width = canvasWidth
  canvasRef.value.height = canvasHeight

  const particleCount = type === 'rain' ? 150 : 80
  for (let i = 0; i < particleCount; i++) {
    if (type === 'rain') {
      particles.push({
        x: Math.random() * canvasWidth,
        y: Math.random() * canvasHeight,
        length: 10 + Math.random() * 15,
        speed: 5 + Math.random() * 8
      })
    } else if (type === 'snow') {
      particles.push({
        x: Math.random() * canvasWidth,
        y: Math.random() * canvasHeight,
        radius: 2 + Math.random() * 4,
        speed: 1 + Math.random() * 3,
        drift: (Math.random() - 0.5) * 0.5
      })
    }
  }
}

function drawRain() {
  if (!ctx) return
  ctx.clearRect(0, 0, canvasWidth, canvasHeight)
  ctx.beginPath()
  ctx.strokeStyle = 'rgba(174, 194, 224, 0.6)'
  ctx.lineWidth = 1.5
  for (let p of particles) {
    ctx.beginPath()
    ctx.moveTo(p.x, p.y)
    ctx.lineTo(p.x - 2, p.y + p.length)
    ctx.stroke()
    p.y += p.speed
    if (p.y > canvasHeight) {
      p.y = -p.length
      p.x = Math.random() * canvasWidth
    }
  }
  animationId = requestAnimationFrame(drawRain)
}

function drawSnow() {
  if (!ctx) return
  ctx.clearRect(0, 0, canvasWidth, canvasHeight)
  ctx.fillStyle = 'rgba(255, 255, 255, 0.8)'
  for (let p of particles) {
    ctx.beginPath()
    ctx.arc(p.x, p.y, p.radius, 0, Math.PI * 2)
    ctx.fill()
    p.y += p.speed
    p.x += p.drift
    if (p.y > canvasHeight) {
      p.y = -p.radius
      p.x = Math.random() * canvasWidth
    }
    if (p.x > canvasWidth) p.x = 0
    if (p.x < 0) p.x = canvasWidth
  }
  animationId = requestAnimationFrame(drawSnow)
}

function startParticleEffect() {
  stopParticleEffect()
  if (!canvasRef.value) return
  ctx = canvasRef.value.getContext('2d')
  const code = props.weatherCode
  let type = null
  if ((code >= 51 && code <= 65) || (code >= 80 && code <= 82)) type = 'rain'
  if ((code >= 71 && code <= 77) || (code >= 85 && code <= 86)) type = 'snow'
  if (!type) return
  initParticles(type)
  if (type === 'rain') drawRain()
  else if (type === 'snow') drawSnow()
}

function stopParticleEffect() {
  if (animationId) {
    cancelAnimationFrame(animationId)
    animationId = null
  }
  if (ctx && canvasRef.value) {
    ctx.clearRect(0, 0, canvasWidth, canvasHeight)
  }
  particles = []
}

function handleResize() {
  if (!showParticleEffect.value) return
  if (canvasRef.value) {
    canvasWidth = window.innerWidth
    canvasHeight = window.innerHeight
    canvasRef.value.width = canvasWidth
    canvasRef.value.height = canvasHeight
    const code = props.weatherCode
    let type = null
    if ((code >= 51 && code <= 65) || (code >= 80 && code <= 82)) type = 'rain'
    if ((code >= 71 && code <= 77) || (code >= 85 && code <= 86)) type = 'snow'
    if (type) initParticles(type)
  }
}

function startLightning() {
  if (lightningInterval) clearInterval(lightningInterval)
  lightningInterval = setInterval(() => {
    if (Math.random() > 0.7) {
      lightningActive.value = true
      setTimeout(() => { lightningActive.value = false }, 150)
    }
  }, 3000)
}

function stopLightning() {
  if (lightningInterval) {
    clearInterval(lightningInterval)
    lightningInterval = null
  }
  lightningActive.value = false
}

watch(() => props.weatherCode, (newCode, oldCode) => {
  stopParticleEffect()
  if (isThunderstorm.value) {
    startLightning()
  } else {
    stopLightning()
  }
  if (showParticleEffect.value) {
    startParticleEffect()
    window.addEventListener('resize', handleResize)
  } else {
    window.removeEventListener('resize', handleResize)
    if (canvasRef.value && ctx) ctx.clearRect(0, 0, canvasWidth, canvasHeight)
  }
})

onMounted(() => {
  if (showParticleEffect.value) {
    startParticleEffect()
    window.addEventListener('resize', handleResize)
  }
  if (isThunderstorm.value) startLightning()
})

onUnmounted(() => {
  stopParticleEffect()
  window.removeEventListener('resize', handleResize)
  stopLightning()
})
</script>

<style scoped>
.background-effect {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  z-index: -2;
  transition: background 0.5s ease;
}

.particle-canvas {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  pointer-events: none;
  z-index: -1;
}

/* Gradients */
.bg-default {
  background: linear-gradient(145deg, #b8d8fc 0%, #e6f0fa 100%);
}
.bg-clear {
  background: linear-gradient(145deg, #3b82f6 0%, #ffe355 80%);
  animation: subtlePulse 8s infinite;
}
.bg-partly-cloudy {
  background: linear-gradient(135deg, #9bb8e0 0%, #d4e7ff 100%);
}
.bg-overcast {
  background: linear-gradient(135deg, #94a3b8 0%, #cbd5e1 100%);
}
.bg-fog {
  background: linear-gradient(135deg, #b9c3ce 0%, #d9e2ec 100%);
}
.bg-rain {
  background: linear-gradient(145deg, #1e3a5f 0%, #4b6e8a 100%);
}
.bg-snow {
  background: linear-gradient(135deg, #d6e4f0 0%, #f8fafc 100%);
}
.bg-thunderstorm {
  background: linear-gradient(145deg, #111827 0%, #374151 100%);
}

.lightning-flash {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(255, 255, 255, 0.9);
  pointer-events: none;
  opacity: 0;
  transition: opacity 0.05s linear;
  z-index: 0;
}
.lightning-flash.active {
  opacity: 0.7;
}

@keyframes subtlePulse {
  0% { background: linear-gradient(145deg, #3b82f6 0%, #ffe355 80%); }
  100% { background: linear-gradient(145deg, #3b82f6 0%, #ffe355 80%); }
}
</style>