<template>
  <div>
    <!-- @seeked="handle" -->

    <video
      ref="videoRef"
      :src="src"
      @loadedmetadata="handleMetadataLoaded"
      @timeupdate="handleTimeUpdate"
      :controls="false"
      width="1920"
    ></video>
  </div>
</template>

<script setup lang="ts">
import { ref, withDefaults, defineProps, watch, defineEmits } from "vue";

const props = withDefaults(
  defineProps<{
    src: string;
    playing: boolean;
    nextFrame: boolean;
    prevFrame: boolean;
  }>(),
  { playing: false }
);

const videoRef = ref();
const emits = defineEmits(["frame-stepped", "current-frame"]);
const currentFrame = ref<number>(1);
const FPS = 12;

let frameDuration = 1 / FPS;

// const handle = (evt: any) => {
//   // console.log(evt);
// };

watch(
  () => props.playing,
  (newVal) => {
    if (videoRef.value) {
      if (newVal) {
        videoRef.value.play();
      } else {
        videoRef.value.pause();
      }
    }
  }
);

watch(
  () => props.nextFrame,
  (isNextFrame) => {
    if (isNextFrame) {
      stepForward();
    }
  }
);
watch(
  () => props.prevFrame,
  (isPrevFrame) => {
    if (isPrevFrame) {
      stepBackward();
    }
  }
);
watch(
  () => currentFrame.value,
  (newFrame) => {
    // console.log("change", newFrame);
    // emits("current-frame", newFrame);
  }
);

const handleMetadataLoaded = () => {
  if (videoRef.value) {
    frameDuration = 1 / videoRef.value.playbackRate / FPS;
  }
};

const handleTimeUpdate = (payload) => {
  // console.log(payload);
  // console.log(Math.floor(videoRef.value.currentTime * FPS) + 1);

  if (videoRef.value) {
    currentFrame.value = Math.floor(videoRef.value.currentTime * FPS) + 1;
    emits("current-frame", currentFrame.value);
  }
};

const stepForward = () => {
  if (videoRef.value) {
    // тестовый вариант рендера кадра
    requestAnimationFrame(() => {
      if (videoRef.value) {
        videoRef.value.currentTime += frameDuration;
        emits("frame-stepped");
        // emits(
        //   "current-frame",
        //   Math.floor(videoRef.value.currentTime * FPS) + 1
        // );
        // currentFrame.value = Math.floor(videoRef.value.currentTime * FPS) + 1;
      }
    });
  }
};
const stepBackward = () => {
  if (videoRef.value) {
    videoRef.value.currentTime -= frameDuration;
    emits("frame-stepped");
    // emits("current-frame", Math.floor(videoRef.value.currentTime * FPS) + 1);
    // currentFrame.value = Math.floor(videoRef.value.currentTime * FPS) + 1;
  }
};

// TODO проверить
// const setFrame = (frameNumber: number) => {
//   const frame = currentFrame.value + frameNumber - 1;
//   if (videoRef.value) {
//     console.log("setFrame call");

//     const targetTime = frame / FPS; // отнимаем 1, так как у нас кадры начинаются с 1, а не с 0
//     videoRef.value.currentTime = targetTime;
//   }
// };
</script>

<style scoped>
/* Здесь можно добавить стили для компонента */
</style>
