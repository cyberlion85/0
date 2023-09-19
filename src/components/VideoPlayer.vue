<template>
  <div>
    <video
      ref="videoRef"
      :src="src"
      @loadedmetadata="handleMetadataLoaded"
      @timeupdate="handleTimeUpdate"
      @seeked="handle"
      controls
      width="480"
    ></video>
    <button @click="stepBackward">Step Backward</button>
    <button @click="stepForward">Step Forward</button>
    <div>Текущее время: {{ currentTime.toFixed(3) }}</div>
    <div>Текущий кадр: {{ currentFrame }}</div>
  </div>
</template>

<script setup lang="ts">
import { ref, withDefaults, defineProps } from "vue";

const props = withDefaults(
  defineProps<{
    src: string;
  }>(),
  {}
);

const videoRef = ref();
// const videoRef = ref<HTMLVideoElement | null>(null);
const currentTime = ref<number>(0);
const currentFrame = ref<number>(1);
const FPS = 12;
// const FPS = 23.98;
// const FPS = 24;
let frameDuration = 1 / FPS;

const handle = (evt: any) => {
  // console.log(evt);
};

const handleMetadataLoaded = () => {
  if (videoRef.value) {
    // frameDuration = 0.042;
    // frameDuration = 1 / FPS;
    frameDuration = 1 / videoRef.value.playbackRate / FPS;
  }
};

const handleTimeUpdate = () => {
  if (videoRef.value) {
    currentTime.value = videoRef.value.currentTime;
    // console.log(currentTime.value * FPS);

    currentFrame.value = Math.floor(currentTime.value * FPS) + 1;
  }
};

const stepForward = () => {
  if (videoRef.value) {
    requestAnimationFrame(() => {
      if (videoRef.value) {
        // videoRef.value.currentTime = 0.05;
        videoRef.value.currentTime += frameDuration;
      }
      console.log(videoRef.value.currentTime);
      // console.log(frameDuration);
    });
  }
};

const stepBackward = () => {
  if (videoRef.value) {
    videoRef.value.currentTime -= frameDuration;
  }
};
</script>

<style scoped>
/* Здесь можно добавить стили для компонента */
</style>
