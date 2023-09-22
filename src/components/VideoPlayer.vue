<template>
  <div>
    <!-- @seeked="handle" -->

    <video
      class="video"
      ref="videoRef"
      :src="src"
      @loadedmetadata="handleMetadataLoaded"
      @timeupdate="handleTimeUpdate"
      :controls="false"
    ></video>
  </div>
</template>

<script setup lang="ts">
import {
  ref,
  withDefaults,
  defineProps,
  watch,
  defineEmits,
  onMounted,
} from "vue";

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
const FPS = 24;

let frameDuration = 1 / FPS;
let previousTime = 0;

const loop = () => {
  if (videoRef.value && props.playing) {
    // console.log(videoRef.value);

    const videoElem = videoRef.value;
    if (videoElem.currentTime !== previousTime) {
      currentFrame.value = Math.floor(videoElem.currentTime * FPS) + 1;
      emits("current-frame", currentFrame.value);
      previousTime = videoElem.currentTime;
    }
  }
  requestAnimationFrame(loop);
};

onMounted(() => {
  requestAnimationFrame(loop);
});

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

const handleMetadataLoaded = () => {
  if (videoRef.value) {
    frameDuration = 1 / videoRef.value.playbackRate / FPS;
  }
};

const handleTimeUpdate = () => {
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
.video {
  width: 100%;
  height: 100%;
}
</style>
