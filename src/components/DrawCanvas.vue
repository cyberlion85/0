<template>
  <div id="canvas_div" style="overflow-x: auto">
    <canvas
      ref="canvasRef"
      width="900"
      height="360"
      @mousedown="handleMouseDown"
      @mousemove="handleMouseMove"
      @mouseup="handleMouseUp"
      @touchstart="handleStart"
      @touchmove="handleMove"
      @touchend="handleEnd"
      @touchcancel="handleCancel"
    ></canvas>
    <button @click="clearArea">Clear Area</button>
    Line width:
    <select v-model="selectedWidth">
      <option value="2">2</option>
      <option value="11">11</option>
      <option value="13">13</option>
      <option value="15">15</option>
    </select>
    Color:
    <select v-model="selectedColor">
      <option value="black">black</option>
      <option value="blue">blue</option>
      <option value="red">red</option>
      <option value="green">green</option>
      <option value="yellow">yellow</option>
      <option value="gray">gray</option>
    </select>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted } from "vue";

const canvasRef = ref<HTMLCanvasElement | null>(null);
const selectedWidth = ref("13");
const selectedColor = ref("blue");
const ongoingTouches = ref<any[]>([]);
let context = null;
let isDrawing = false;
let x = 0;
let y = 0;
let offsetX = 0;
let offsetY = 0;

onMounted(() => {
  if (canvasRef.value) {
    context = canvasRef.value.getContext("2d");
    const rect = canvasRef.value.getBoundingClientRect();
    offsetX = rect.left;
    offsetY = rect.top;
  }
});

function handleMouseDown(e) {
  x = e.offsetX;
  y = e.offsetY;
  isDrawing = true;
}

function handleMouseMove(e) {
  if (isDrawing) {
    drawLine(context, x, y, e.offsetX, e.offsetY);
    x = e.offsetX;
    y = e.offsetY;
  }
}

function handleMouseUp(e) {
  if (isDrawing) {
    drawLine(context, x, y, e.offsetX, e.offsetY);
    x = 0;
    y = 0;
    isDrawing = false;
  }
}

// ... остальные функции по аналогии с вашим кодом ...

function drawLine(context, x1, y1, x2, y2) {
  context.beginPath();
  context.strokeStyle = selectedColor.value;
  context.lineWidth = selectedWidth.value;
  context.lineJoin = "round";
  context.moveTo(x1, y1);
  context.lineTo(x2, y2);
  context.closePath();
  context.stroke();
}

function clearArea() {
  if (context) {
    context.setTransform(1, 0, 0, 1, 0, 0);
    context.clearRect(0, 0, context.canvas.width, context.canvas.height);
  }
}
</script>

<style scoped>
#canvas_div {
  text-align: center;
  display: block;
  margin-left: auto;
  margin-right: auto;
}
canvas {
  border: 2px solid black;
}
</style>
