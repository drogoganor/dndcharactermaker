<template>
  <div id="app">
    <DnD />
  </div>
</template>

<script>
// eslint-disable-next-line
import pako from './pako'
import DnD from './components/DnD.vue'

export default {
  name: 'app',
  components: {
    DnD
  }
}
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 20px;
}
</style>
