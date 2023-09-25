<template>
  <div id="svg_container">
    <button @click="mode = 'draw'">Draw Mode</button>
    <button @click="mode = 'move'">Move Mode</button>
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
        @mouseover="hoverPath(index)"
        @mouseout="unhoverPath"
        :class="{ highlight: index === hoveredPathIndex }"
      />
    </svg>
  </div>
</template>

<script setup lang="ts">
import { ref, reactive } from "vue";

let mode = ref("draw"); // "draw" or "move"
let isDrawing = false;
let isMoving = false;
let currentPath: string[] = [];
let pathStrings: string[] = reactive([]);
let selectedPathIndex = ref<number | null>(null); // Make it reactive
let hoveredPathIndex = ref<number | null>(null);
let lastX: number | null = null;
let lastY: number | null = null;
let initialX = 0;
let initialY = 0;
let strokeWidth = ref(5);
let deltaX = 0;
let deltaY = 0;

const smoothingFactor = 0.1;

const svgRef = ref<SVGSVGElement | null>(null);

const handleMouseDown = (e: MouseEvent) => {
  if (mode.value === "draw") {
    isDrawing = true;
    lastX = e.offsetX;
    lastY = e.offsetY;
    currentPath.push(`M${lastX} ${lastY}`);
    pathStrings.push("");
  } else if (mode.value === "move") {
    isMoving = true;
    selectedPathIndex.value = hoveredPathIndex.value;
    initialX = e.offsetX;
    initialY = e.offsetY;
  }
};

const handleMouseMove = (e: MouseEvent) => {
  if (mode.value === "draw" && isDrawing && lastX !== null && lastY !== null) {
    let newX = e.offsetX;
    let newY = e.offsetY;

    lastX = lastX + (newX - lastX) * smoothingFactor;
    lastY = lastY + (newY - lastY) * smoothingFactor;

    currentPath.push(`L${lastX.toFixed(2)} ${lastY.toFixed(2)}`);
    pathStrings[pathStrings.length - 1] = currentPath.join(" ");
  } else if (
    mode.value === "move" &&
    isMoving &&
    selectedPathIndex.value !== null
  ) {
    deltaX = e.offsetX - initialX;
    deltaY = e.offsetY - initialY;
    initialX = e.offsetX;
    initialY = e.offsetY;

    let movedPath = pathStrings[selectedPathIndex.value].replace(
      /([ML])([\d.]+) ([\d.]+)/g,
      (match, command, x, y) => {
        x = parseFloat(x) + deltaX;
        y = parseFloat(y) + deltaY;
        return `${command}${x} ${y}`;
      }
    );
    pathStrings[selectedPathIndex.value] = movedPath;
  }
};

const handleMouseUp = () => {
  isDrawing = false;
  isMoving = false;
  lastX = null;
  lastY = null;
  currentPath = [];
};

const hoverPath = (index: number) => {
  if (mode.value === "move") {
    hoveredPathIndex.value = index;
  }
};

const unhoverPath = () => {
  hoveredPathIndex.value = null;
};
</script>

<style scoped>
.svg-canvas {
  border: 1px solid black;
}
.highlight {
  stroke: blue;
}
</style>
