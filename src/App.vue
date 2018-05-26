<template>
  <div id="app">
    <Tetris/>
  </div>
</template>

<script>
import Tetris from './components/Tetris'

export default {
  name: 'App',
  components: {
    Tetris
  }
}
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}
</style>
