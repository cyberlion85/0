<template>
  <div class="container" ref="divMain" @wheel="onWheel" @mouseup="onMouseUp">
    <div
      class="draggable-container"
      ref="draggableContainer"
      @contextmenu.prevent
      @mousedown.right="onMouseDown"
      @mousemove="onMouseMove"
      @mouseup="onMouseUp"
    >
      <slot></slot>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, defineEmits } from "vue";

const draggableContainer = ref<HTMLElement | null>(null);
const divMain = ref<HTMLElement | null>(null);
let scale = 1;
const factor = 0.05;
const max_scale = 4;
let previousX: number, previousY: number;
let offsetX = 0;
let offsetY = 0;

const emits = defineEmits(["zoomChange"]);

const onMouseDown = (event: MouseEvent) => {
  if (draggableContainer.value) {
    previousX = event.pageX;
    previousY = event.pageY;
    draggableContainer.value.style.cursor = "grab";
  }
};

const onMouseMove = (event: MouseEvent) => {
  if (event.buttons === 2) {
    let dragX = event.pageX - previousX;
    let dragY = event.pageY - previousY;
    previousX = event.pageX;
    previousY = event.pageY;
    offsetX += dragX;
    offsetY += dragY;

    if (draggableContainer.value) {
      draggableContainer.value.style.transform = `translate(${offsetX}px, ${offsetY}px) scale(${scale}, ${scale})`;
    }
  }
};

const onMouseUp = () => {
  if (draggableContainer.value) {
    draggableContainer.value.style.cursor = "default";
  }
};

const onWheel = (e: WheelEvent) => {
  e.preventDefault();
  let delta = e.deltaY;
  delta = Math.max(-1, Math.min(1, delta));

  const prevScale = scale;

  scale -= delta * factor * scale;
  scale = Math.max(1, Math.min(max_scale, scale));

  // console.log(scale);
  emits("zoomChange", scale);

  if (divMain.value) {
    const x = e.pageX - divMain.value.getBoundingClientRect().left;
    const y = e.pageY - divMain.value.getBoundingClientRect().top;
    offsetX = x - (x - offsetX) * (scale / prevScale);
    offsetY = y - (y - offsetY) * (scale / prevScale);

    if (draggableContainer.value) {
      draggableContainer.value.style.transform = `translate(${offsetX}px, ${offsetY}px) scale(${scale}, ${scale})`;
    }
  }
};
</script>

<style scoped>
.container {
  width: 1280px;
  height: 534px;
  overflow: hidden;
  position: relative;
  /* margin: auto; */
}

.draggable-container {
  position: absolute;
  transform-origin: 0 0;
  width: auto;
  height: auto;
}
</style>
