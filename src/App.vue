<script setup>
import { onBeforeUnmount, onMounted, ref } from 'vue'

const canvas = ref(null)
let animationFrame = 0

const heartPoint = (t) => {
  const x = 16 * Math.sin(t) ** 3
  const y =
    13 * Math.cos(t) -
    5 * Math.cos(2 * t) -
    2 * Math.cos(3 * t) -
    Math.cos(4 * t)

  return { x, y: -y }
}

onMounted(() => {
  const element = canvas.value
  const context = element.getContext('2d')
  const particles = []
  const twinkles = []
  let width = 0
  let height = 0
  let pixelRatio = 1

  const resize = () => {
    pixelRatio = Math.min(window.devicePixelRatio || 1, 2)
    width = window.innerWidth
    height = window.innerHeight
    element.width = width * pixelRatio
    element.height = height * pixelRatio
    element.style.width = `${width}px`
    element.style.height = `${height}px`
    context.setTransform(pixelRatio, 0, 0, pixelRatio, 0, 0)
  }

  const createParticles = () => {
    particles.length = 0
    const count = Math.min(1100, Math.max(520, Math.floor(width * height / 1450)))

    for (let index = 0; index < count; index += 1) {
      const t = Math.random() * Math.PI * 2
      const point = heartPoint(t)
      const depth = Math.random() ** 0.58
      const jitter = (1 - depth) * 0.9

      particles.push({
        x: point.x + (Math.random() - 0.5) * jitter,
        y: point.y + (Math.random() - 0.5) * jitter,
        depth,
        size: 0.7 + Math.random() * 2.2,
        phase: Math.random() * Math.PI * 2,
        speed: 0.7 + Math.random() * 1.6,
        hue: 336 + Math.random() * 24,
      })
    }
  }

  const createTwinkles = () => {
    twinkles.length = 0

    for (let index = 0; index < 80; index += 1) {
      twinkles.push({
        x: Math.random() * width,
        y: Math.random() * height,
        size: Math.random() * 1.7 + 0.4,
        phase: Math.random() * Math.PI * 2,
        speed: Math.random() * 0.9 + 0.25,
      })
    }
  }

  const reset = () => {
    resize()
    createParticles()
    createTwinkles()
  }

  const draw = (time) => {
    const seconds = time / 1000
    const centerX = width / 2
    const centerY = height / 2 + Math.min(height * 0.035, 28)
    const scaleBase = Math.min(width, height) / 39
    const beat =
      1 +
      Math.max(0, Math.sin(seconds * Math.PI * 1.75)) ** 8 * 0.12 +
      Math.sin(seconds * Math.PI * 3.5) * 0.012

    context.globalCompositeOperation = 'source-over'
    context.fillStyle = 'rgba(5, 4, 13, 0.22)'
    context.fillRect(0, 0, width, height)

    const halo = context.createRadialGradient(centerX, centerY, 10, centerX, centerY, scaleBase * 18)
    halo.addColorStop(0, 'rgba(255, 48, 128, 0.28)')
    halo.addColorStop(0.45, 'rgba(164, 33, 255, 0.08)')
    halo.addColorStop(1, 'rgba(5, 4, 13, 0)')
    context.fillStyle = halo
    context.fillRect(0, 0, width, height)

    context.globalCompositeOperation = 'lighter'

    for (const star of twinkles) {
      const alpha = 0.18 + Math.sin(seconds * star.speed + star.phase) ** 2 * 0.5
      context.beginPath()
      context.fillStyle = `rgba(255, 245, 250, ${alpha})`
      context.arc(star.x, star.y, star.size, 0, Math.PI * 2)
      context.fill()
    }

    for (const particle of particles) {
      const pulse = Math.sin(seconds * particle.speed + particle.phase) * 0.6
      const drift = Math.cos(seconds * particle.speed * 0.8 + particle.phase) * 0.55
      const scale = scaleBase * (0.56 + particle.depth * 0.52) * beat
      const x = centerX + (particle.x + drift * (1 - particle.depth)) * scale
      const y = centerY + (particle.y + pulse * (1 - particle.depth)) * scale
      const alpha = 0.38 + particle.depth * 0.55
      const radius = particle.size * (0.72 + particle.depth * 1.35) * beat

      context.beginPath()
      context.fillStyle = `hsla(${particle.hue}, 100%, ${64 + particle.depth * 18}%, ${alpha})`
      context.shadowBlur = 12 + particle.depth * 22
      context.shadowColor = `hsla(${particle.hue}, 100%, 62%, 0.95)`
      context.arc(x, y, radius, 0, Math.PI * 2)
      context.fill()
    }

    context.shadowBlur = 0
    context.globalCompositeOperation = 'screen'
    context.strokeStyle = 'rgba(255, 225, 237, 0.18)'
    context.lineWidth = 1.2
    context.beginPath()

    for (let index = 0; index <= 240; index += 1) {
      const point = heartPoint((index / 240) * Math.PI * 2)
      const x = centerX + point.x * scaleBase * 0.98 * beat
      const y = centerY + point.y * scaleBase * 0.98 * beat

      if (index === 0) {
        context.moveTo(x, y)
      } else {
        context.lineTo(x, y)
      }
    }

    context.closePath()
    context.stroke()
    animationFrame = requestAnimationFrame(draw)
  }

  reset()
  window.addEventListener('resize', reset)
  animationFrame = requestAnimationFrame(draw)

  onBeforeUnmount(() => {
    window.removeEventListener('resize', reset)
    cancelAnimationFrame(animationFrame)
  })
})
</script>

<template>
  <main class="app">
    <canvas ref="canvas" class="heart-canvas" aria-label="粒子跳动爱心"></canvas>
  </main>
</template>
