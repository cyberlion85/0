<template>
  <!-- <img alt="Vue logo" src="./assets/logo.png" /> -->
  <div v-if="false">
    <VideoPlayer src="/12.mov" />
  </div>
  <DrawCanvas />
</template>

<script lang="ts">
import { defineComponent } from "vue";
// import HelloWorld from "./components/HelloWorld.vue";
import VideoPlayer from "./components/VideoPlayer.vue";
import DrawCanvas from "./components/DrawCanvas.vue";

export default defineComponent({
  name: "App",
  components: {
    // HelloWorld,
    VideoPlayer,
    DrawCanvas,
  },
});
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
