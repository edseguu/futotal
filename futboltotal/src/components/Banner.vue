<script setup>
import { ref, computed, onMounted, onBeforeUnmount } from 'vue'

// Load all PNGs placed under src/banner/* automatically
// Use eager + as:url to get final asset URLs from Vite
const logos = ref([])
const phrases = [
  'Juego limpio, pasión total',
  'La Reta: donde nacen campeones',
  'Fútbol total — potencia y corazón',
  '¡Que ruede el balón!'
]
// Build a mixed track: logos interleaved with phrases so text appears between logos
const trackItems = computed(() => {
  const items = []
  const logosArr = logos.value || []
  if (logosArr.length === 0) return items
  let p = 0
  for (let i = 0; i < logosArr.length; i++) {
    items.push({ type: 'logo', src: logosArr[i] })
    // Insert a phrase every 2 logos
    if ((i + 1) % 2 === 0) {
      items.push({ type: 'phrase', text: phrases[p % phrases.length] })
      p++
    }
  }
  return items
})

onMounted(() => {
  try {
    // Prefer assets in src/assets/banner, but also support src/banner for backwards compatibility
    const fromAssets = import.meta.glob('../assets/banner/*.{png,PNG}', { eager: true, as: 'url' })
    const fromLegacy = import.meta.glob('../banner/*.{png,PNG}', { eager: true, as: 'url' })
    const urls = [...Object.values(fromAssets), ...Object.values(fromLegacy)]
    // de-dupe while preserving order
    const seen = new Set()
    logos.value = urls.filter(u => (seen.has(u) ? false : (seen.add(u), true)))
  } catch (e) {
    logos.value = []
  }

})

onBeforeUnmount(() => {
  // nothing
})
</script>

<template>
  <div class="banner-root" aria-hidden="true">
    <div class="glass"></div>
    <div class="track-mask"></div>

    <div class="slider" :class="{ empty: logos.length === 0 }">
      <!-- Duplicate the row for seamless loop -->
      <div class="row">
        <template v-if="trackItems.length > 0">
          <template v-for="(it, i) in trackItems" :key="`a-`+i">
            <img v-if="it.type==='logo'" class="logo" :src="it.src" alt="logo" draggable="false" />
            <div v-else class="phrase-chip">{{ it.text }}</div>
          </template>
        </template>
        <template v-else>
          <div class="placeholder">Agrega logos .png en <code>src/assets/banner</code></div>
        </template>
      </div>
      <div class="row">
        <template v-if="trackItems.length > 0">
          <template v-for="(it, i) in trackItems" :key="`b-`+i">
            <img v-if="it.type==='logo'" class="logo" :src="it.src" alt="logo" draggable="false" />
            <div v-else class="phrase-chip">{{ it.text }}</div>
          </template>
        </template>
        <template v-else>
          <div class="placeholder">Agrega logos .png en <code>src/assets/banner</code></div>
        </template>
      </div>
    </div>
  </div>
</template>

<style scoped>
.banner-root {
  position: fixed;
  left: 50%;
  bottom: 0;
  transform: translateX(-50%);
  width: clamp(600px, 50vw, 1400px);
  height: clamp(60px, 10vh, 120px);
  z-index: 400; /* below overlays but above content */
  pointer-events: none; /* decorative, avoid blocking clicks */
  overflow: hidden;
}

/* Glass layer */
.glass {
  position: absolute;
  inset: 0;
  background: rgba(255,255,255,0.06);
  backdrop-filter: blur(8px) saturate(140%);
  -webkit-backdrop-filter: blur(8px) saturate(140%);
  border-top: 1px solid rgba(255,255,255,0.15);
}

/* subtle gradient mask at edges */
.track-mask {
  pointer-events: none;
  position: absolute;
  inset: 0;
  background: linear-gradient(90deg,
    rgba(0,0,0,0.35) 0%,
    rgba(0,0,0,0.0) 10%,
    rgba(0,0,0,0.0) 90%,
    rgba(0,0,0,0.35) 100%);
  mix-blend-mode: multiply;
}

.slider {
  position: absolute;
  top: 0;
  bottom: 0;
  left: clamp(16px, 4vw, 64px);
  right: clamp(16px, 4vw, 64px);
  display: flex;
  align-items: center;
  gap: 0;
}

.row {
  display: flex;
  align-items: center;
  gap: clamp(32px, 4vw, 72px);
  padding: 0 clamp(28px, 4vw, 56px);
  animation: scrollX 60s linear infinite;
  will-change: transform;
}

/* Two rows side-by-side for a seamless loop */
/* Remove extra margin between rows to avoid a visible jump at the seam */
.row + .row { margin-left: 0; }

@keyframes scrollX {
  0% { transform: translate3d(0,0,0); }
  100% { transform: translate3d(-100%,0,0); }
}

.logo {
  height: clamp(28px, 6vh, 64px);
  width: auto;
  object-fit: contain;
  filter: drop-shadow(0 2px 8px rgba(0,0,0,0.35));
  opacity: 0.95;
  user-select: none;
  -webkit-user-drag: none;
}

.phrase-chip {
  pointer-events: none;
  color: #ffffff;
  font-weight: 900;
  font-size: clamp(12px, 2vw, 18px);
  letter-spacing: 0;
  text-align: left;
  white-space: nowrap;
  padding: 6px 12px;
  border-radius: 999px;
  background: rgba(255,255,255,0.08);
  border: 1px solid rgba(255,255,255,0.18);
  box-shadow: 0 2px 10px rgba(0,0,0,0.25);
  backdrop-filter: blur(6px) saturate(120%);
  -webkit-backdrop-filter: blur(6px) saturate(120%);
}

.placeholder {
  color: rgba(255,255,255,0.7);
  font-weight: 700;
  font-size: clamp(12px, 2.2vw, 18px);
}

/* removed fixed corner phrase; phrases now scroll between logos */

/* Subtle visibility contrast to keep minimalist */
.banner-root::after {
  content: '';
  position: absolute;
  inset: 0;
  box-shadow: inset 0 6px 30px rgba(0,0,0,0.25);
  pointer-events: none;
}

/* Ultra-wide (pantallas grandes) */
@media (min-width: 2400px) {
  .banner-root {
    /* make banner larger on very large screens */
    width: clamp(1200px, 70vw, 2400px);
    height: clamp(80px, 12vh, 160px);
  }
}
</style>
