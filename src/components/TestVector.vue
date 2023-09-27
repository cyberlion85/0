<template>
  <div id="svg_container">
    <div>Before: {{ currentPath.length }}</div>
    <div>After: {{ simplifiedPathRes }}</div>
    <button @click="clearCanvas">Clear</button>
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
        :stroke="selectedColor"
        :stroke-width="selectedStrokeWidth"
        fill="none"
      />
    </svg>
  </div>
</template>

<script setup lang="ts">
import { ref, reactive } from "vue";

interface Point {
  x: number;
  y: number;
}

const perpendicularDistance = (
  point: Point,
  lineStart: Point,
  lineEnd: Point
): number => {
  const dx = lineEnd.x - lineStart.x;
  const dy = lineEnd.y - lineStart.y;

  const numerator = Math.abs(
    dy * point.x -
      dx * point.y +
      lineEnd.x * lineStart.y -
      lineEnd.y * lineStart.x
  );
  const denominator = Math.sqrt(Math.pow(dy, 2) + Math.pow(dx, 2));

  return numerator / denominator;
};

const ramerDouglasPeucker = (points: Point[], epsilon: number): Point[] => {
  let dmax = 0;
  let index = 0;

  for (let i = 1; i < points.length - 1; i++) {
    const d = perpendicularDistance(
      points[i],
      points[0],
      points[points.length - 1]
    );

    if (d > dmax) {
      index = i;
      dmax = d;
    }
  }

  let result: Point[] = [];

  if (dmax > epsilon) {
    const recResults1 = ramerDouglasPeucker(
      points.slice(0, index + 1),
      epsilon
    );
    const recResults2 = ramerDouglasPeucker(points.slice(index), epsilon);

    result = recResults1.slice(0, recResults1.length - 1).concat(recResults2);
  } else {
    result = [points[0], points[points.length - 1]];
  }

  return result;
};

let isDrawing = false;
let lastX: number | null = null;
let lastY: number | null = null;
let currentPath: Point[] = [];
let pathStrings: string[] = reactive([]);
const svgRef = ref<SVGSVGElement | null>(null);
const selectedStrokeWidth = ref(5);
const selectedColor = ref("#000000");
let simplifiedPathRes = ref();

const epsilon = 0.5;
const smoothingFactor = 0.2;

const handleMouseDown = (e: MouseEvent) => {
  isDrawing = true;
  lastX = e.offsetX;
  lastY = e.offsetY;
  currentPath = [{ x: lastX, y: lastY }];
  pathStrings.push(`M${lastX.toFixed(2)} ${lastY.toFixed(2)}`);
};

const handleMouseMove = (e: MouseEvent) => {
  if (isDrawing && lastX !== null && lastY !== null) {
    const newX = lastX + (e.offsetX - lastX) * smoothingFactor;
    const newY = lastY + (e.offsetY - lastY) * smoothingFactor;

    lastX = newX;
    lastY = newY;

    currentPath.push({ x: newX, y: newY });

    const simplifiedPath = ramerDouglasPeucker(currentPath, epsilon);

    if (simplifiedPath.length) simplifiedPathRes.value = simplifiedPath.length;
    pathStrings[pathStrings.length - 1] = simplifiedPath
      .map(
        ({ x, y }, idx) =>
          `${idx === 0 ? "M" : "L"}${x.toFixed(2)} ${y.toFixed(2)}`
      )
      .join(" ");
  }
};

const handleMouseUp = () => {
  isDrawing = false;
  lastX = null;
  lastY = null;
};

const clearCanvas = () => {
  pathStrings.length = 0;
  currentPath = [];
};
</script>

<style scoped>
.svg-canvas {
  border: 1px solid black;
}
</style>
