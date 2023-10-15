<template>
  <div>
    <div class="video-draw">
      <CanvasControls
        class="canvas-controls"
        @draw="mode = 'draw'"
        @draw-line="mode = 'drawLine'"
        @draw-arrow="mode = 'drawArrow'"
        @erase="mode = 'erase'"
        @move="mode = 'move'"
        @delete="mode = 'delete'"
        @undo="isUndo = true"
        @clear="clear()"
        @color-change="(color) => (selectedColor = color)"
        @stroke-width-change="(width) => (selectedStrokeWidth = width)"
        @smoothing-factor-change="(evt) => (childSmoothingFactor = evt)"
        @epsilon-change="(evt) => (childEpsilon = evt)"
        @eraser-size-change="(evt) => (childEaserSize = evt)"
      ></CanvasControls>

      <draggableElement
        class="dragable-element"
        @zoom-change="(zoom) => (zoomScale = zoom)"
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
            @update:frames-with-data="(data) => (framesWithSketch = data)"
            :currentFrame="currentFrame"
            ref="canvasRef"
            class="canvas"
            :video-height="videoHeight"
            :video-width="videoWidth"
            :mode="mode"
            :is-undo="isUndo"
            :selected-color="selectedColor"
            :selected-stroke-width="selectedStrokeWidth"
            :smoothing-factor="childSmoothingFactor"
            :epsilon="childEpsilon"
            :eraser-size="childEaserSize"
            :zoom-scale="zoomScale"
            @reset-undo-click="isUndo = false"
          />
          <DrawCanvas
            v-if="!isVector"
            @update:frame-to-image-data="(data) => (framesWithSketch = data)"
            :currentFrame="currentFrame"
            ref="canvasRef"
            class="canvas"
          /></div
      ></draggableElement>
    </div>

    <div class="controls">
      <TimeLine
        @selected-frame="(frame) => (selectedFrame = frame)"
        @clicked-play="isPlaying = !isPlaying"
        :framesWithSketch="framesWithSketch"
        :playingFrame="currentFrame"
        :totalFrames="totalFrames"
        :isPlaying="isPlaying"
      />
      <button @click="isPrevFrame = true">Step Backward</button>
      <button @click="isNextFrame = true">Step Forward</button>
    </div>
  </div>
</template>

<script setup lang="ts">
import { onBeforeUnmount, onMounted, ref } from "vue";
import VideoPlayer from "./VideoPlayer.vue";
import DrawCanvas from "./DrawCanvas.vue";
import DrawVector from "./DrawVector.vue";
import draggableElement from "./draggableElement.vue";
import TimeLine from "./TimeLine.vue";
import CanvasControls from "./CanvasControls.vue";

// const test = (a: any) => {
//   console.log(a);
// };

const onKeydown = (event: KeyboardEvent) => {
  // console.log(event);

  if (event.key === "ArrowLeft") {
    isPrevFrame.value = true;
  } else if (event.key === "ArrowRight") {
    isNextFrame.value = true;
  }
};

onMounted(() => {
  window.addEventListener("keydown", onKeydown);
});

onBeforeUnmount(() => {
  console.log("onBeforeUnmount");

  window.removeEventListener("keydown", onKeydown);
});

const mode = ref<string>("draw");
// const isErase = ref(false);
const isUndo = ref(false);

const videoPlayerRef = ref(null);
const isVector = ref(true);
const canvasRef = ref<ExtendedHTMLCanvasElement | null>(null);
interface ExtendedHTMLCanvasElement extends HTMLCanvasElement {
  clearScreen: () => void;
}

const isPlaying = ref(false);
const currentFrame = ref(0);
const totalFrames = ref(0);
const isNextFrame = ref(false);
const isPrevFrame = ref(false);
const framesWithSketch = ref([]);
const selectedFrame = ref<number | null>(null);
const selectedColor = ref("");
const selectedStrokeWidth = ref("3");

// Параметры сглаживания и упрощения
const childSmoothingFactor = ref(NaN);
const childEpsilon = ref(NaN);
// Параметры ластика
const childEaserSize = ref(NaN);

const zoomScale = ref(1);

const filename = "/sample_movie.mp4";
const videoHeight = 534;
const videoWidth = 1280;
// const filename = "/24.mov";
// const videoHeight = 768;
// const videoWidth = 1366;
const clear = () => {
  canvasRef.value?.clearScreen();
};
</script>

<style scoped>
.video-draw {
  /* border: 5px solid red; */
}
.video-draw-container {
  position: relative;
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
  position: fixed;
  bottom: 0;
  left: 0;
  width: 100%;
  z-index: 1;
}
</style>
