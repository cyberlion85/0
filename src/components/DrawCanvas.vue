<template>
  <div id="canvas_div" style="overflow-x: auto">
    <!--  -->
    <canvas
      ref="canvasRef"
      class="canvas"
      width="1920"
      height="1080"
      @mousedown="handleMouseDown"
      @mousemove="handleMouseMove"
      @mouseup="handleMouseUp"
    ></canvas>
    <!-- <button @click="clearArea">Clear Area</button>
    Line width:
    <select v-model="selectedWidth">
      <option :value="3">3</option>
      <option :value="5">5</option>
      <option :value="7">7</option>
      <option :value="9">9</option>
    </select>
    Color:
    <select v-model="selectedColor">
      <option value="black">black</option>
      <option value="blue">blue</option>
      <option value="red">red</option>
      <option value="green">green</option>
      <option value="yellow">yellow</option>
      <option value="gray">gray</option>
    </select> -->
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted } from "vue";

const canvasRef = ref<HTMLCanvasElement | null>(null);
const selectedWidth = ref(8);
const selectedColor = ref("blue");
let context: CanvasRenderingContext2D | null = null;
let isDrawing = false;
let x = 0;
let y = 0;

onMounted(() => {
  if (canvasRef.value) {
    context = canvasRef.value.getContext("2d");
    // const rect = canvasRef.value.getBoundingClientRect();
  }
});

function handleMouseDown(e: MouseEvent) {
  x = e.offsetX;
  y = e.offsetY;
  isDrawing = true;
}

function handleMouseMove(e: MouseEvent) {
  if (isDrawing && context) {
    drawLine(context, x, y, e.offsetX, e.offsetY);
    x = e.offsetX;
    y = e.offsetY;
  }
}

function handleMouseUp(e: MouseEvent) {
  if (isDrawing && context) {
    drawLine(context, x, y, e.offsetX, e.offsetY);
    x = 0;
    y = 0;
    isDrawing = false;
  }
}

function drawLine(
  context: CanvasRenderingContext2D,
  x1: number,
  y1: number,
  x2: number,
  y2: number
) {
  context.beginPath();
  context.imageSmoothingEnabled = false;
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
  /* border: 3px solid rgb(255, 0, 0); */
}
.canvas {
  /* width: 1920;
  height: 1080; */
  padding: 0px;
}
</style>
