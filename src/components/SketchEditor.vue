<template>
  <div>
    <draggableElement
      ><div class="video-draw-container">
        <VideoPlayer
          ref="videoPlayerRef"
          :next-frame="isNextFrame"
          :prev-frame="isPrevFrame"
          :playing="isPlaying"
          :selected-frame="selectedFrame"
          @frame-stepped="(isNextFrame = false), (isPrevFrame = false)"
          @current-frame="(frameNum) => (currentFrame = frameNum)"
          class="player"
          src="/24.mov"
        />
        <DrawCanvas
          @update:frame-to-image-data="(data) => (framesWithSketch = data)"
          :currentFrame="currentFrame"
          ref="canvasRef"
          class="canvas"
        /></div
    ></draggableElement>

    <br />
    <div class="controls">
      <button @click="isPlaying = true">Play</button>
      <button @click="isPlaying = false">Stop</button>
      <button @click="isPrevFrame = true">Step Backward</button>
      <button @click="isNextFrame = true">Step Forward</button>

      <div>Текущий кадр: {{ currentFrame }}</div>
    </div>
    <TimeLine
      @selected-frame="(frame) => (selectedFrame = frame)"
      :framesWithSketch="framesWithSketch"
      :playingFrame="currentFrame"
      :totalFrames="500"
    />
  </div>
</template>

<script setup lang="ts">
import { ref } from "vue";
import VideoPlayer from "./VideoPlayer.vue";
import DrawCanvas from "./DrawCanvas.vue";
import draggableElement from "./draggableElement.vue";
import TimeLine from "./TimeLine.vue";

const videoPlayerRef = ref(null);

const canvasRef = ref<null>(null);

const isPlaying = ref(false);
const currentFrame = ref(0);
const isNextFrame = ref(false);
const isPrevFrame = ref(false);
const framesWithSketch = ref([]);
const selectedFrame = ref<number | null>(null);
</script>

<style scoped>
.video-draw-container {
  position: relative;
  width: 1366px;
  height: 768px;
  overflow: hidden;
  margin: auto;
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
  /* z-index: 2; */
}
</style>
