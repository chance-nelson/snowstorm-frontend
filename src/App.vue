<template>
  <div id="app">
    <link href="https://fonts.googleapis.com/icon?family=Material+Icons" rel="stylesheet" crossorigin="anonymous">
    <Player streamSource="http://34.70.62.142/stream/stream.mp3" streamMimeType="audio/mpeg"/>
  </div>
</template>

<script>
//import HelloWorld from './components/HelloWorld.vue'
import Player from './components/Player.vue'

export default {
  name: 'app',
  components: {
    Player
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
  margin-top: 60px;
}
</style>
