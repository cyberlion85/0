<template>
  <!-- <div class="video-draw-container">
    <VideoPlayer class="player" src="/12.mov" />
    <DrawCanvas class="canvas" />
  </div>
  <div class="controls">
    <button>Step Backward</button>
    <button>Step Forward</button>
    <div>Текущее время: 111</div>
    <div>Текущий кадр:222</div>
  </div> -->
  <SketchEditor />
</template>

<script lang="ts">
import { defineComponent } from "vue";
import SketchEditor from "./components/SketchEditor.vue";
// import VideoPlayer from "./components/VideoPlayer.vue";
// import DrawCanvas from "./components/DrawCanvas.vue";

export default defineComponent({
  name: "App",
  components: {
    // eslint-disable-next-line vue/no-unused-components
    // VideoPlayer,
    // eslint-disable-next-line vue/no-unused-components
    // DrawCanvas,
    SketchEditor,
  },
});
</script>

<style scoped>
/* #app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
} */
.video-draw-container {
  /* border: 3px solid green; */
  position: relative;
  /* width: auto; */
  /* height: auto; */
  width: 1920px;
  height: 1080px;
  overflow: hidden;
}

.player,
.canvas {
  position: absolute;
  top: 0;
  left: 0;
  /* width: 100%; */
  /* height: 100%; */
}

.player {
  z-index: 1;
}

.canvas {
  z-index: 2;
}
.controls {
  border: 1px solid red;
}
</style>
