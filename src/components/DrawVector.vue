<template>
  <div id="svg_container">
    <button @click="mode = 'draw'">Draw</button>
    <button @click="mode = 'move'">Move</button>
    <button @click="mode = 'drawLine'">Draw Line</button>
    <select v-model="selectedStrokeWidth">
      <option value="1">1px</option>
      <option value="3">3px</option>
      <option value="5">5px</option>
      <option value="7">7px</option>
      <option value="10">10px</option>
    </select>
    <input type="color" v-model="selectedColor" />
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
        :stroke="colors[index] || selectedColor"
        :stroke-width="strokeWidths[index] || selectedStrokeWidth"
        fill="none"
        @mouseover="hoverPath(index)"
        @mouseout="unhoverPath"
        :class="{ highlight: index === hoveredPathIndex }"
      />
    </svg>
  </div>
</template>

<script setup lang="ts">
import {
  ref,
  reactive,
  withDefaults,
  defineProps,
  defineEmits,
  defineExpose,
  watch,
} from "vue";

const props = withDefaults(
  defineProps<{
    currentFrame: number;
  }>(),
  { currentFrame: 0 }
);

const emits = defineEmits(["update:frameToImageData"]);

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
let deltaX = 0;
let deltaY = 0;

const smoothingFactor = 0.1;

const svgRef = ref<SVGSVGElement | null>(null);
const svgFramesData: Record<number, string[]> = {};

let selectedStrokeWidth = ref(5); // Текущая выбранная толщина линии
let strokeWidths: number[] = reactive([]); // Толщина для каждого пути

let selectedColor = ref("#e1da09"); // Текущий выбранный цвет
let colors: string[] = reactive([]); // Цвет для каждого пути

const saveSvgData = () => {
  svgFramesData[props.currentFrame] = [...pathStrings];

  emits("update:frameToImageData", Object.keys(svgFramesData).map(Number));
};

const loadSvgData = () => {
  const data = svgFramesData[props.currentFrame];
  if (data) {
    pathStrings.length = 0;
    data.forEach((path) => pathStrings.push(path));
    // console.log("Loaded SVG data for frame:", props.currentFrame);
  } else {
    pathStrings.length = 0; // Clear the paths if no data for the frame
    // console.log("No SVG data for frame:", props.currentFrame);
  }
};
defineExpose({
  saveSvgData,
  loadSvgData,
});

watch(
  () => props.currentFrame,
  () => {
    loadSvgData();
  }
);

const handleMouseDown = (e: MouseEvent) => {
  lastX = e.offsetX;
  lastY = e.offsetY;

  if (mode.value === "draw") {
    isDrawing = true;
    currentPath.push(`M${lastX} ${lastY}`);
    pathStrings.push("");
    strokeWidths.push(selectedStrokeWidth.value);
    colors.push(selectedColor.value);
  } else if (mode.value === "drawLine") {
    isDrawing = true;
    pathStrings.push(`M${lastX} ${lastY}`);
    strokeWidths.push(selectedStrokeWidth.value);
    colors.push(selectedColor.value);
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
  } else if (mode.value === "drawLine" && isDrawing) {
    let endX = e.offsetX;
    let endY = e.offsetY;
    pathStrings[pathStrings.length - 1] = `M${lastX} ${lastY} L${endX} ${endY}`;
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
        if (x >= 0 && x <= 1366 && y >= 0 && y <= 768) {
          return `${command}${x} ${y}`;
        } else isMoving = false;

        return `${command}${parseFloat(x) - deltaX} ${parseFloat(y) - deltaY}`;
      }
    );
    pathStrings[selectedPathIndex.value] = movedPath;
    // console.log(movedPath);
  }
};

const handleMouseUp = () => {
  isDrawing = false;
  isMoving = false;
  lastX = null;
  lastY = null;
  currentPath = [];
  saveSvgData();
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
  border: 1px solid rgb(94, 255, 0);
}
.highlight {
  stroke: blue;
  cursor: pointer;
}
</style>
