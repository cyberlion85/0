<template>
  <img alt="Vue logo" src="./assets/logo.png" />
  <!-- <HelloWorld msg="Welcome to Your Vue.js + TypeScript App" /> -->
  <VideoPlayer src="/12.mov" />
  <!-- <VideoPlayer src="/che_v031_out_srgb.mp4" /> -->
</template>

<script lang="ts">
import { defineComponent } from "vue";
// import HelloWorld from "./components/HelloWorld.vue";
import VideoPlayer from "./components/VideoPlayer.vue";

export default defineComponent({
  name: "App",
  components: {
    // HelloWorld,
    VideoPlayer,
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
