<template>
  <div id="canvas_div" style="overflow-x: auto">
    <canvas
      ref="canvasRef"
      class="canvas"
      width="1366"
      height="768"
      @mousedown="handleMouseDown"
      @mousemove="handleMouseMove"
      @mouseup="handleMouseUp"
    ></canvas>
  </div>
</template>

<script setup lang="ts">
import {
  ref,
  onMounted,
  defineExpose,
  withDefaults,
  defineProps,
  watch,
  defineEmits,
} from "vue";

const props = withDefaults(
  defineProps<{
    currentFrame: number;
  }>(),
  { currentFrame: 0 }
);

const emits = defineEmits(["update:frameToImageData"]);

const canvasRef = ref<HTMLCanvasElement | null>(null);
const selectedWidth = ref(6);
const selectedColor = ref("blue");
let context: CanvasRenderingContext2D | null = null;
let isDrawing = false;
let x: number | null = null;
let y: number | null = null;
const frameToImageData: Record<number, string> = {};
let lastTime = Date.now();

let smoothLineWidth = 1; // Инициализация сглаженной толщины
const smoothingFactor = 0.05; // Фактор сглаживания, значение от 0 до 1

onMounted(() => {
  if (canvasRef.value) {
    context = canvasRef.value.getContext("2d");
  }
});
watch(
  () => props.currentFrame,
  () => {
    // Вызовите loadFromBase64 каждый раз, когда меняется currentFrame
    loadFromBase64();
  }
);

const handleMouseDown: (e: MouseEvent) => void = (e) => {
  x = e.offsetX;
  y = e.offsetY;
  isDrawing = true;
};

const handleMouseMove: (e: MouseEvent) => void = (e) => {
  if (isDrawing) {
    const currentTime = Date.now();
    const deltaTime = currentTime - lastTime;

    if (deltaTime === 0 || x === null || y === null) return;

    const dx = e.offsetX - x;
    const dy = e.offsetY - y;

    const speed = Math.sqrt(dx * dx + dy * dy) / deltaTime;

    const targetLineWidth = Math.min(20, Math.max(2, speed * 2));
    // console.log(targetLineWidth);

    // Применяем экспоненциальное сглаживание
    smoothLineWidth += (targetLineWidth - smoothLineWidth) * smoothingFactor;

    if (isDrawing && context) {
      selectedWidth.value = smoothLineWidth;
      drawLine(context, x, y, e.offsetX, e.offsetY);
      x = e.offsetX;
      y = e.offsetY;
    }

    lastTime = currentTime;
    if (isDrawing) {
      lastTime = currentTime;
    }
  }
};

const handleMouseUp: (e: MouseEvent) => void = (e) => {
  isDrawing = false;
  saveToBase64();
};

const drawLine: (
  context: CanvasRenderingContext2D,
  x1: number,
  y1: number,
  x2: number,
  y2: number
) => void = (context, x1, y1, x2, y2) => {
  context.beginPath();
  context.imageSmoothingEnabled = false;
  context.strokeStyle = selectedColor.value;
  context.lineWidth = selectedWidth.value;
  context.lineJoin = "round";
  context.moveTo(x1, y1);
  context.lineTo(x2, y2);
  context.closePath();
  context.stroke();
};

const clearArea: () => void = () => {
  if (context) {
    context.setTransform(1, 0, 0, 1, 0, 0);
    context.clearRect(0, 0, context.canvas.width, context.canvas.height);
  }
};

const saveToBase64: () => void = () => {
  if (!canvasRef.value) return;
  const dataURL = canvasRef.value.toDataURL("image/png");
  frameToImageData[props.currentFrame] = dataURL;
  emits("update:frameToImageData", Object.keys(frameToImageData).map(Number));
  console.log(
    `Image for frame ${props.currentFrame} saved to temporary variable`
  );
};

const loadFromBase64: () => void = () => {
  if (!context) return;
  clearArea();
  const imgData = frameToImageData[props.currentFrame];
  if (imgData) {
    const img = new Image();
    img.onload = () => {
      if (context) {
        context.drawImage(img, 0, 0);
      }
    };
    img.src = imgData;
  } else {
    // console.log(`No image data for frame ${props.currentFrame}`);
  }
};
defineExpose({
  saveToBase64,
  loadFromBase64,
});
</script>

<style scoped></style>
