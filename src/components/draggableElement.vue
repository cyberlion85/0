<template>
  <div class="container" ref="divMain">
    <div class="draggable-container" ref="draggableContainer">
      <slot></slot>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted } from "vue";

const draggableContainer = ref<HTMLElement | null>(null);
const divMain = ref<HTMLElement | null>(null);
let scale = 1;
const factor = 0.05;
const max_scale = 4;
let previousX: number, previousY: number;
let offsetX = 0;
let offsetY = 0;

onMounted(() => {
  draggableContainer.value?.addEventListener(
    "contextmenu",
    (event: MouseEvent) => {
      event.preventDefault();
    }
  );

  draggableContainer.value?.addEventListener(
    "mousedown",
    (event: MouseEvent) => {
      if (event.button === 2) {
        previousX = event.pageX;
        previousY = event.pageY;
        if (draggableContainer.value) {
          draggableContainer.value.style.cursor = "grab"; // Устанавливаем курсор на grabbing при начале перетаскивания
        }
      }
    }
  );
  document.addEventListener("mouseup", () => {
    if (draggableContainer.value) {
      draggableContainer.value.style.cursor = "default"; // Устанавливаем курсор обратно на default после завершения перетаскивания
    }
  });

  draggableContainer.value?.addEventListener(
    "mousemove",
    (event: MouseEvent) => {
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
    }
  );

  divMain.value?.addEventListener("wheel", (e: WheelEvent) => {
    e.preventDefault();
    let delta = e.deltaY;
    delta = Math.max(-1, Math.min(1, delta));

    const prevScale = scale;

    scale -= delta * factor * scale;
    scale = Math.max(1, Math.min(max_scale, scale));

    if (divMain.value) {
      // Находим координаты точки, относительно которой происходит масштабирование
      const x = e.pageX - divMain.value.getBoundingClientRect().left;
      const y = e.pageY - divMain.value.getBoundingClientRect().top;
      // Обновляем смещения
      offsetX = x - (x - offsetX) * (scale / prevScale);
      offsetY = y - (y - offsetY) * (scale / prevScale);

      if (draggableContainer.value) {
        draggableContainer.value.style.transform = `translate(${offsetX}px, ${offsetY}px) scale(${scale}, ${scale})`;
      }
    }
  });
});
</script>

<style scoped>
.container {
  width: 80vw;
  height: 80vh;
  overflow: hidden;
  position: relative;
  margin: auto;
  /* border: 1px solid red; */
}

.draggable-container {
  /* border: 3px solid blue; */
  position: absolute;

  transform-origin: 0 0;
  width: auto;
  height: auto;
}
</style>
