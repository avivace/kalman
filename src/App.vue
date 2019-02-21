<template>
  <div id="app">
    <Kalman/>
  </div>
</template>

<script>
import Kalman from './components/Kalman.vue'

export default {
  name: 'app',
  components: {
    Kalman
  }
}
</script>

<style>
@import url('https://rsms.me/inter/inter.css');
html { font-family: 'Inter', sans-serif; }
@supports (font-variation-settings: normal) {
  html { font-family: 'Inter var', sans-serif; }
}
#app {
  font-family: 'Inter','Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}

</style>
