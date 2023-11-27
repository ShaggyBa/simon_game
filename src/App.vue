<template>
  <div id="app">
    <Game />
  </div>
</template>

<script>
import Game from './components/Game/Game.vue'

export default {
  name: 'App',
  components: {
    Game
  }
}
</script>

<style>
html, body {
  height: 100%;
  margin: 0;
  padding: 0;
}
body {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  min-height: 100%;

}
</style>
