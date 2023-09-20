<template>
  <div class="video-draw-container">
    <VideoPlayer
      ref="videoPlayerRef"
      :next-frame="isNextFrame"
      :prev-frame="isPrevFrame"
      :playing="isPlaying"
      @frame-stepped="(isNextFrame = false), (isPrevFrame = false)"
      @current-frame="(frameNum) => (currentFrame = frameNum)"
      class="player"
      src="/12.mov"
    />
    <DrawCanvas class="canvas" />
  </div>
  <br />
  <div class="controls">
    <button @click="isPlaying = true">Play</button>
    <button @click="isPlaying = false">Stop</button>
    <button @click="isPrevFrame = true">Step Backward</button>
    <button @click="isNextFrame = true">Step Forward</button>
    <!-- <div>Текущее время: 111</div> -->
    <div>Текущий кадр: {{ currentFrame }}</div>
  </div>
</template>

<script setup lang="ts">
import { ref } from "vue";
import VideoPlayer from "./VideoPlayer.vue";
import DrawCanvas from "./DrawCanvas.vue";

const videoPlayerRef = ref(null);
const isPlaying = ref(false);
let currentFrame = ref(0);
let isNextFrame = ref(false);
let isPrevFrame = ref(false);
</script>

<style scoped>
.video-draw-container {
  position: relative;
  width: 1920px;
  height: 1080px;
  overflow: hidden;
}

.player,
.canvas {
  position: absolute;
  top: 0;
  left: 0;
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
