<template>
  <div
    class="container"
    @mousemove="handleMouseMove"
    @mouseup.right="handleMouseUp"
    @contextmenu.prevent
  >
    <div class="box" ref="box" @mousedown.right.prevent="handleMouseDown">
      <slot></slot>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, onUnmounted } from "vue";

const box = ref<HTMLElement | null>(null);
let isDragging = false;
let offsetX = 0;
let offsetY = 0;

const handleMouseDown = (e: MouseEvent) => {
  if (e.button === 2 && box.value) {
    isDragging = true;

    const boxStyle = getComputedStyle(box.value);
    if (boxStyle) {
      offsetX = e.clientX - parseInt(boxStyle.left);
      offsetY = e.clientY - parseInt(boxStyle.top);
    }
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
  /* width: 1200px; */
  /* height: 800px; */
  width: 80vw;
  height: 80vh;
  position: relative;
  /* border: 1px solid blue; */
  margin: auto;
}

.box {
  width: 100px;
  height: 100px;
  background-color: red;
  position: absolute;
  left: 0;
  top: 0;
  z-index: 0;
}
</style>
