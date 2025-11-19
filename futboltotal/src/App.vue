<script setup>
import Bracket from './components/Bracket.vue'
import Banner from './components/Banner.vue'
import { ref, onMounted } from 'vue'

// assign logo path at runtime to avoid Vite transforming the literal into a static import
const logoSrc = ref(null)
const logoVisible = ref(true)

function onLogoError() {
  logoVisible.value = false
}

onMounted(() => {
  // set at runtime so missing file won't break compilation
  logoSrc.value = '/LARETA_logo.png'
})
</script>

<template>
  <div class="app-root">
    <Bracket />
    <Banner />
  </div>
</template>

<style scoped>
.app-root{
  font-family: system-ui, -apple-system, 'Segoe UI', Roboto, 'Helvetica Neue', Arial;
  padding: 0;
  margin: 0;
  display:flex;
  flex-direction:column;
  align-items:center;
  justify-content:center;
  min-height: 100vh;
  width: 100vw;
  overflow: hidden;
}
</style>
