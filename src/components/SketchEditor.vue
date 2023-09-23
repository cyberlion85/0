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
    <DrawCanvas
      @update:frame-to-image-data="(data) => (framesWithSketch = data)"
      :currentFrame="currentFrame"
      ref="canvasRef"
      class="canvas"
    />
  </div>
  <br />
  <div class="controls">
    <button @click="isPlaying = true">Play</button>
    <button @click="isPlaying = false">Stop</button>
    <button @click="isPrevFrame = true">Step Backward</button>
    <button @click="isNextFrame = true">Step Forward</button>
    <!-- <button style="border: 1px solid blue" @click="save">Save</button> -->
    <!-- <button style="border: 1px solid blue" @click="load">Load</button> -->
    <!-- <div>Текущее время: 111</div> -->
    <div>Текущий кадр: {{ currentFrame }}</div>
  </div>
  <TimeLine
    :framesWithSketch="framesWithSketch"
    :playingFrame="currentFrame"
    :totalFrames="500"
  />
</template>

<script setup lang="ts">
import { ref } from "vue";
import VideoPlayer from "./VideoPlayer.vue";
import DrawCanvas from "./DrawCanvas.vue";
import TimeLine from "./TimeLine.vue";

const videoPlayerRef = ref(null);
interface CanvasComponentMethods {
  saveToBase64: () => void;
  loadFromBase64: () => void;
}

const canvasRef = ref<CanvasComponentMethods | null>(null);

const isPlaying = ref(false);
let currentFrame = ref(0);
let isNextFrame = ref(false);
let isPrevFrame = ref(false);
let framesWithSketch = ref([]);

const save = () => {
  canvasRef.value?.saveToBase64();
};
const load = () => {
  canvasRef.value?.loadFromBase64();
};
</script>

<style scoped>
.video-draw-container {
  position: relative;
  width: 1366px;
  height: 768px;
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
  /* border: 1px solid red; */
}
</style>
