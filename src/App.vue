<template>
  <TelegramVisualizer />
  <HelloWorld v-if="false" msg="Welcome to Your Vue.js + TypeScript App" />
</template>

<script lang="ts">
import { defineComponent } from 'vue'
import HelloWorld from './components/HelloWorld.vue'
import TelegramVisualizer from './components/TelegramVisualizer.vue'

export default defineComponent({
  components: { HelloWorld, TelegramVisualizer }
})
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
