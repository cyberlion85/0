<template>
  <div>
    <div>Before: {{ currentPath.length }}</div>
    <div>After: {{ simplifiedPathRes }}</div>
    <button @click="clearCanvas">Clear</button>
    <button @click="rasterize">Rasterize</button>
    <button @click="showSvg = !showSvg">Toggle SVG</button>
    <button @click="toggleDrawingMode">
      {{ drawingMode === "draw" ? "Eraser Mode" : "Draw Mode" }}
    </button>
    <div v-if="!showSvg">
      Eraser Size: <input type="range" v-model="eraserSize" min="5" max="50" />
    </div>
    <div id="svg_container">
      <svg
        v-if="showSvg"
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
      <canvas
        v-show="!showSvg"
        ref="canvasRef"
        class="canvas"
        width="1366"
        height="768"
        @mousedown.left="handleCanvasMouseDown"
        @mousemove="handleCanvasMouseMove"
        @mouseup.left="handleCanvasMouseUp"
      ></canvas>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, reactive } from "vue";

interface Point {
  x: number;
  y: number;
}

const canvasRef = ref<HTMLCanvasElement>();
const svgRef = ref<SVGSVGElement>();
const showSvg = ref(true);
let isDrawing = false;
let lastX: number | null = null;
let lastY: number | null = null;
let currentPath: Point[] = [];
let pathStrings: string[] = reactive([]);
const selectedStrokeWidth = ref(5);
const selectedColor = ref("#000000");
let simplifiedPathRes = ref();
const epsilon = 0.1;
const smoothingFactor = 1;

const eraserSize = ref(20);
const drawingMode = ref("draw");

const rasterize = () => {
  const svgElement = svgRef.value;
  const canvasElement = canvasRef.value;
  if (canvasElement && svgElement) {
    const ctx = canvasElement.getContext("2d");
    const data = new XMLSerializer().serializeToString(svgElement);
    const svg = new Blob([data], { type: "image/svg+xml;charset=utf-8" });
    const url = URL.createObjectURL(svg);
    const img = new Image();

    img.onload = () => {
      ctx.drawImage(img, 0, 0);
      URL.revokeObjectURL(url);
    };

    img.src = url;
  }
};

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

const handleMouseDown = (e: MouseEvent) => {
  isDrawing = true;
  lastX = e.offsetX;
  lastY = e.offsetY;
  currentPath = [{ x: lastX, y: lastY }];
  pathStrings.push(`M${lastX.toFixed(2)} ${lastY.toFixed(2)}`);
};

const handleMouseMove = (e: MouseEvent) => {
  if (!isDrawing || lastX === null || lastY === null) return;

  const newX = lastX + (e.offsetX - lastX) * smoothingFactor;
  const newY = lastY + (e.offsetY - lastY) * smoothingFactor;
  lastX = newX;
  lastY = newY;
  currentPath.push({ x: newX, y: newY });

  const currentPoint = { x: e.offsetX, y: e.offsetY };
  const smoothedPath = smoothPath(currentPath, 0.1, currentPoint);
  const simplifiedPath = ramerDouglasPeucker(smoothedPath, epsilon);

  if (simplifiedPath.length) simplifiedPathRes.value = simplifiedPath.length;
  pathStrings[pathStrings.length - 1] = simplifiedPath
    .map(
      ({ x, y }, idx) =>
        `${idx === 0 ? "M" : "L"}${x.toFixed(2)} ${y.toFixed(2)}`
    )
    .join(" ");
};

const smoothPath = (
  path: Point[],
  alpha: number,
  currentPoint: Point
): Point[] => {
  if (path.length < 2) return path;
  const smoothed: Point[] = [path[0]];

  for (let i = 1; i < path.length; i++) {
    const prev = smoothed[i - 1];
    const curr = path[i];
    const smoothedX = alpha * curr.x + (1 - alpha) * prev.x;
    const smoothedY = alpha * curr.y + (1 - alpha) * prev.y;
    smoothed.push({ x: smoothedX, y: smoothedY });
  }
  smoothed.push(currentPoint);
  return smoothed;
};

const handleMouseUp = () => {
  isDrawing = false;
  lastX = null;
  lastY = null;
};

const clearCanvas = () => {
  pathStrings.length = 0;
  currentPath = [];
  const canvas = canvasRef.value;
  if (canvas) {
    const ctx = canvas.getContext("2d");
    if (ctx) {
      ctx.clearRect(0, 0, canvas.width, canvas.height);
    }
  }
};

const toggleDrawingMode = () => {
  drawingMode.value = drawingMode.value === "draw" ? "erase" : "draw";
};

const handleCanvasMouseDown = (e: MouseEvent) => {
  if (drawingMode.value === "draw") {
    isDrawing = true;
  } else if (drawingMode.value === "erase") {
    isDrawing = true;
    erase(e);
  }
};

const handleCanvasMouseMove = (e: MouseEvent) => {
  if (isDrawing && drawingMode.value === "erase") {
    erase(e);
  }
};

const handleCanvasMouseUp = () => {
  isDrawing = false;
};

const erase = (e: MouseEvent) => {
  const canvas = canvasRef.value;
  if (!canvas) return;

  const ctx = canvas.getContext("2d");
  if (!ctx) return;

  const x = e.offsetX - eraserSize.value / 2;
  const y = e.offsetY - eraserSize.value / 2;

  ctx.clearRect(x, y, eraserSize.value, eraserSize.value);
};
</script>

<style scoped>
.svg-canvas,
.canvas {
  border: 1px solid black;
}
</style>
