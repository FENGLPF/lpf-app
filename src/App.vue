<script setup>
import { onBeforeUnmount, onMounted, ref } from 'vue'

const canvasRef = ref(null)
let animationFrame = 0

onMounted(() => {
  const canvas = canvasRef.value
  const context = canvas.getContext('2d')
  const isMobile = /android|webos|iphone|ipad|ipod|blackberry|iemobile|opera mini/i.test(
    (navigator.userAgent || navigator.vendor || window.opera).toLowerCase(),
  )
  const coefficient = isMobile ? 0.5 : 1
  const random = Math.random
  const particles = []
  const pointsOrigin = []
  const targetPoints = []
  const traceCount = isMobile ? 20 : 50
  const step = isMobile ? 0.3 : 0.1
  const config = {
    traceK: 0.4,
    timeDelta: 0.01,
  }

  let width = 0
  let height = 0
  let time = 0

  const heartPosition = (radian) => [
    Math.sin(radian) ** 3,
    -(15 * Math.cos(radian) - 5 * Math.cos(2 * radian) - 2 * Math.cos(3 * radian) - Math.cos(4 * radian)),
  ]

  const scaleAndTranslate = (position, scaleX, scaleY, deltaX, deltaY) => [
    deltaX + position[0] * scaleX,
    deltaY + position[1] * scaleY,
  ]

  const resize = () => {
    width = canvas.width = coefficient * window.innerWidth
    height = canvas.height = coefficient * window.innerHeight
    canvas.style.width = `${window.innerWidth}px`
    canvas.style.height = `${window.innerHeight}px`
    context.fillStyle = 'rgba(0,0,0,1)'
    context.fillRect(0, 0, width, height)
  }

  const pulse = (scaleX, scaleY) => {
    for (let index = 0; index < pointsOrigin.length; index += 1) {
      targetPoints[index] = [
        scaleX * pointsOrigin[index][0] + width / 2,
        scaleY * pointsOrigin[index][1] + height / 2,
      ]
    }
  }

  for (let radian = 0; radian < Math.PI * 2; radian += step) {
    pointsOrigin.push(scaleAndTranslate(heartPosition(radian), 210, 13, 0, 0))
  }

  for (let radian = 0; radian < Math.PI * 2; radian += step) {
    pointsOrigin.push(scaleAndTranslate(heartPosition(radian), 150, 9, 0, 0))
  }

  for (let radian = 0; radian < Math.PI * 2; radian += step) {
    pointsOrigin.push(scaleAndTranslate(heartPosition(radian), 90, 5, 0, 0))
  }

  const heartPointsCount = pointsOrigin.length

  resize()

  for (let index = 0; index < heartPointsCount; index += 1) {
    const x = random() * width
    const y = random() * height
    const trace = []

    for (let traceIndex = 0; traceIndex < traceCount; traceIndex += 1) {
      trace[traceIndex] = { x, y }
    }

    particles[index] = {
      vx: 0,
      vy: 0,
      speed: random() + 5,
      q: Math.floor(random() * heartPointsCount),
      direction: 2 * (index % 2) - 1,
      force: 0.2 * random() + 0.7,
      color: `hsla(0,${Math.floor(40 * random() + 60)}%,${Math.floor(60 * random() + 20)}%,.3)`,
      trace,
    }
  }

  const loop = () => {
    const pulseValue = -Math.cos(time)
    pulse((1 + pulseValue) * 0.5, (1 + pulseValue) * 0.5)
    time += (Math.sin(time) < 0 ? 9 : pulseValue > 0.8 ? 0.2 : 1) * config.timeDelta

    context.fillStyle = 'rgba(0,0,0,.1)'
    context.fillRect(0, 0, width, height)

    for (let index = particles.length; index--;) {
      const particle = particles[index]
      const target = targetPoints[particle.q]
      const dx = particle.trace[0].x - target[0]
      const dy = particle.trace[0].y - target[1]
      const distance = Math.sqrt(dx * dx + dy * dy)
      const safeDistance = distance || 1

      if (distance < 10) {
        if (random() > 0.95) {
          particle.q = Math.floor(random() * heartPointsCount)
        } else {
          if (random() > 0.99) {
            particle.direction *= -1
          }

          particle.q += particle.direction
          particle.q %= heartPointsCount

          if (particle.q < 0) {
            particle.q += heartPointsCount
          }
        }
      }

      particle.vx += (-dx / safeDistance) * particle.speed
      particle.vy += (-dy / safeDistance) * particle.speed
      particle.trace[0].x += particle.vx
      particle.trace[0].y += particle.vy
      particle.vx *= particle.force
      particle.vy *= particle.force

      for (let traceIndex = 0; traceIndex < particle.trace.length - 1;) {
        const current = particle.trace[traceIndex]
        const next = particle.trace[++traceIndex]
        next.x -= config.traceK * (next.x - current.x)
        next.y -= config.traceK * (next.y - current.y)
      }

      context.fillStyle = particle.color

      for (let traceIndex = 0; traceIndex < particle.trace.length; traceIndex += 1) {
        context.fillRect(particle.trace[traceIndex].x, particle.trace[traceIndex].y, 1, 1)
      }
    }

    context.fillStyle = 'rgba(255,255,255,1)'

    for (let index = Math.min(traceCount + 13, targetPoints.length); index--;) {
      context.fillRect(targetPoints[index][0], targetPoints[index][1], 2, 2)
    }

    animationFrame = window.requestAnimationFrame(loop)
  }

  window.addEventListener('resize', resize)
  loop()

  onBeforeUnmount(() => {
    window.removeEventListener('resize', resize)
    window.cancelAnimationFrame(animationFrame)
  })
})
</script>

<template>
  <canvas ref="canvasRef" class="heart-canvas"></canvas>
</template>
