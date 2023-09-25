<template>
  <div id="svg_container">
    <svg
      ref="svgRef"
      class="svg-canvas"
      width="1366"
      height="768"
      @mousedown.left="handleMouseDown"
      @mousemove="handleMouseMove"
      @mouseup.left="handleMouseUp"
    >
      <path
        v-for="(pathString, index) in pathStrings"
        :key="index"
        :d="pathString"
        stroke="black"
        :stroke-width="strokeWidth"
        fill="none"
      />
    </svg>
  </div>
</template>

<script setup lang="ts">
import { ref, reactive } from "vue";

let isDrawing = false;
let currentPath: string[] = [];
let pathStrings: string[] = reactive([]);
let lastX: number | null = null;
let lastY: number | null = null;
let strokeWidth = ref(5); // Начальная толщина линии

const smoothingFactor = 0.2; // Устанавливаем фактор сглаживания, 0 - 1

const svgRef = ref<SVGSVGElement | null>(null);

const handleMouseDown = (e: MouseEvent) => {
  isDrawing = true;
  lastX = e.offsetX;
  lastY = e.offsetY;
  currentPath.push(`M${lastX} ${lastY}`);
  pathStrings.push(""); // Добавить пустую строку, которая будет заполнена при рисовании
};

const handleMouseMove = (e: MouseEvent) => {
  if (isDrawing && lastX !== null && lastY !== null) {
    lastX = lastX + (e.offsetX - lastX) * smoothingFactor;
    lastY = lastY + (e.offsetY - lastY) * smoothingFactor;
    currentPath.push(`L${lastX.toFixed(2)} ${lastY.toFixed(2)}`);
    pathStrings[pathStrings.length - 1] = currentPath.join(" "); // Обновляем последний элемент
  }
};

const handleMouseUp = () => {
  if (isDrawing) {
    isDrawing = false;
    lastX = null;
    lastY = null;
    currentPath = [];
  }
};
</script>

<style scoped>
.svg-canvas {
  border: 1px solid black;
}
</style>
