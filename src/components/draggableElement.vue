<template>
  <div
    class="container"
    @mousemove="handleMouseMove"
    @mouseup.right="handleMouseUp"
    @contextmenu.prevent
  >
    <!-- <SketchEditor
      class="box"
      ref="box"
      @mousedown.right.prevent="handleMouseDown"
    ></SketchEditor> -->
    <div class="box" ref="box" @mousedown.right.prevent="handleMouseDown"></div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, onUnmounted } from "vue";
import SketchEditor from "./SketchEditor.vue";

const box = ref<HTMLElement | null>(null);
let isDragging = false;
let offsetX = 0;
let offsetY = 0;

const handleMouseDown = (e: MouseEvent) => {
  if (e.button === 2 && box.value) {
    isDragging = true;
    offsetX = e.clientX - (parseInt(box.value.style.left) || 0);
    offsetY = e.clientY - (parseInt(box.value.style.top) || 0);
  }
};

const handleMouseMove = (e: MouseEvent) => {
  if (isDragging && box.value) {
    box.value.style.left = e.clientX - offsetX + "px";
    box.value.style.top = e.clientY - offsetY + "px";
  }
};

const handleMouseUp = () => {
  isDragging = false;
};

const handleContextMenu = (e: Event) => {
  e.preventDefault();
};

onMounted(() => {
  window.addEventListener("contextmenu", handleContextMenu);
});

onUnmounted(() => {
  window.removeEventListener("contextmenu", handleContextMenu);
});
</script>

<style scoped>
.container {
  width: 100vw;
  height: 100vh;
  position: relative;
}

.box {
  width: 100px;
  height: 100px;
  background-color: red;
  position: absolute;
  left: 0;
  top: 0;
}
</style>
