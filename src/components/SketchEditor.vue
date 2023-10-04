<template>
  <div>
    <div>
      <!-- mode {{ mode }} -->

      <CanvasControls
        @draw="mode = 'draw'"
        @draw-line="mode = 'drawLine'"
        @draw-arrow="mode = 'drawArrow'"
        @move="mode = 'move'"
        @delete="mode = 'delete'"
        @erase="isErase = !isErase"
      ></CanvasControls>
    </div>
    <draggableElement
      ><div
        class="video-draw-container"
        :style="{ height: videoHeight + 'px', width: videoWidth + 'px' }"
      >
        <VideoPlayer
          ref="videoPlayerRef"
          :next-frame="isNextFrame"
          :prev-frame="isPrevFrame"
          :startPlay="isPlaying"
          :selected-frame="selectedFrame"
          @total-frames="(total) => (totalFrames = total)"
          @frame-stepped="(isNextFrame = false), (isPrevFrame = false)"
          @current-frame="(frameNum) => (currentFrame = frameNum)"
          class="player"
          :src="filename"
        />
        <DrawVector
          v-if="isVector"
          @update:frame-to-image-data="(data) => (framesWithSketch = data)"
          :currentFrame="currentFrame"
          ref="canvasRef"
          class="canvas"
          :video-height="videoHeight"
          :video-width="videoWidth"
          :mode="mode"
          :is-erase="isErase"
        />
        <DrawCanvas
          v-if="!isVector"
          @update:frame-to-image-data="(data) => (framesWithSketch = data)"
          :currentFrame="currentFrame"
          ref="canvasRef"
          class="canvas"
        /></div
    ></draggableElement>

    <div class="controls">
      <!-- <button @click="isPlaying = true">Play</button> -->
      <!-- <button @click="isPlaying = false">Stop</button> -->
      <button @click="isPrevFrame = true">Step Backward</button>
      <button @click="isNextFrame = true">Step Forward</button>

      <div>Текущий кадр: {{ currentFrame }}</div>
    </div>
    <TimeLine
      @selected-frame="(frame) => (selectedFrame = frame)"
      @clicked-play="isPlaying = !isPlaying"
      :framesWithSketch="framesWithSketch"
      :playingFrame="currentFrame"
      :totalFrames="totalFrames"
      :isPlaying="isPlaying"
    />
  </div>
</template>

<script setup lang="ts">
import { ref } from "vue";
import VideoPlayer from "./VideoPlayer.vue";
import DrawCanvas from "./DrawCanvas.vue";
import DrawVector from "./DrawVector.vue";
import draggableElement from "./draggableElement.vue";
import TimeLine from "./TimeLine.vue";
import CanvasControls from "./CanvasControls.vue";

const mode = ref<string>("draw");
const isErase = ref(false);

const videoPlayerRef = ref(null);
const isVector = ref(true);
const canvasRef = ref<null>(null);

const isPlaying = ref(false);
const currentFrame = ref(0);
const totalFrames = ref(0);
const isNextFrame = ref(false);
const isPrevFrame = ref(false);
const framesWithSketch = ref([]);
const selectedFrame = ref<number | null>(null);

const filename = "/sample_movie.mp4";
const videoHeight = 534;
const videoWidth = 1280;
// const filename = "/24.mov";
// const videoHeight = 768;
// const videoWidth = 1366;
</script>

<style scoped>
.video-draw-container {
  position: relative;
  /* width: 1366px;
  height: 768px; */
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
