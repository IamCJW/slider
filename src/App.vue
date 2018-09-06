<template>
  <div id="app">
    <slider/>
  </div>
</template>

<script>
import slider from './components/slider'

export default {
  name: 'App',
  components: {
    slider
  }
}
</script>

<style>
  html,body{
    padding: 0;
    margin: 0;
  }
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}
</style>
