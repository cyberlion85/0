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
    selectedFrame: number | null;
  }>(),
  { playing: false }
);

const videoRef = ref();
const emits = defineEmits(["frame-stepped", "current-frame"]);
const currentFrame = ref<number>(1);
const FPS = 24;

// const frameDuration = parseFloat((1 / FPS).toFixed(5));
const frameDuration = 1 / FPS;
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
  emits("current-frame", currentFrame.value);
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

watch(
  () => props.selectedFrame,
  (frame) => {
    if (frame) {
      setFrame(frame);
    }
  }
);

const handleMetadataLoaded = () => {
  // if (videoRef.value) {
  //   frameDuration = 1 / videoRef.value.playbackRate / FPS;
  // }
};

const handleTimeUpdate = () => {
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
        setFrame(currentFrame.value + 1);
        emits("frame-stepped");
      }
    });
  }
};
const stepBackward = () => {
  if (videoRef.value) {
    setFrame(currentFrame.value - 1);
    emits("frame-stepped");
  }
};

// TODO проверить
const setFrame = (frameNumber: number) => {
  if (videoRef.value) {
    const targetTime = frameNumber / FPS;
    // вычитаем половину длинн фрейма чтобы попасть в центр кадра
    videoRef.value.currentTime = targetTime - frameDuration / 2;
  }
};
</script>

<style scoped>
.video {
  width: 100%;
  height: 100%;
}
</style>
