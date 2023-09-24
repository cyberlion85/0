<template>
  <!-- <SketchEditor /> -->
  <draggableElement />
</template>

<script setup lang="ts">
import SketchEditor from "./components/SketchEditor.vue";
import draggableElement from "./components/draggableElement.vue";
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
