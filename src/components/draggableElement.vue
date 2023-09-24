<template>
  <div
    class="container"
    @mousemove="handleMouseMove"
    @mouseup.right="handleMouseUp"
    @wheel="handleWheel"
    @contextmenu.prevent
  >
    <div
      class="draggable-container"
      ref="draggableContainer"
      @mousedown.right.prevent="handleMouseDown"
      :style="{ transform: `scale(${scale})` }"
    >
      <slot></slot>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, onUnmounted } from "vue";

const draggableContainer = ref<HTMLElement | null>(null);
let isDragging = false;
let offsetX = 0;
let offsetY = 0;
let scale = ref(1); // начальный масштаб
const minScale = 0.1; // минимальный масштаб
const maxScale = 8; // максимальный масштаб

const handleMouseDown = (e: MouseEvent) => {
  if (e.button === 2 && draggableContainer.value) {
    isDragging = true;

    const containerStyle = getComputedStyle(draggableContainer.value);
    if (containerStyle) {
      offsetX = e.clientX - parseInt(containerStyle.left);
      offsetY = e.clientY - parseInt(containerStyle.top);
    }
  }
};

const handleMouseMove = (e: MouseEvent) => {
  if (isDragging && draggableContainer.value) {
    draggableContainer.value.style.left = e.clientX - offsetX + "px";
    draggableContainer.value.style.top = e.clientY - offsetY + "px";
  }
};

const handleMouseUp = () => {
  isDragging = false;
};

const handleWheel = (e: WheelEvent) => {
  e.preventDefault();
  const delta = e.deltaY > 0 ? 0.9 : 1.1;
  const newScale = scale.value * delta;

  if (newScale >= minScale && newScale <= maxScale) {
    scale.value = newScale;
  }
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
  width: 80vw;
  height: 80vh;
  position: relative;
  margin: auto;
}

.draggable-container {
  position: absolute;
  left: 0;
  top: 0;
  z-index: 0;
  transition: transform 0.1s ease;
}
</style>
