<template>
  <div id="svg_container">
    <button @click="(mode = 'draw'), (selectedStrokeWidth = 5)">Draw</button>
    <button @click="mode = 'move'">Move</button>
    <button @click="(mode = 'drawLine'), (selectedStrokeWidth = 3)">
      Line
    </button>
    <button @click="(mode = 'drawArrow'), (selectedStrokeWidth = 3)">
      Arrow
    </button>
    <button @click="mode = 'text'">Text</button>

    <button @click="undoLastPath">Undo</button>
    <button @click="clearCanvas">Clear</button>
    <select v-model="selectedStrokeWidth">
      <option value="1">1px</option>
      <option value="3">3px</option>
      <option value="5">5px</option>
      <option value="7">7px</option>
      <option value="10">10px</option>
    </select>
    <input type="color" v-model="selectedColor" />
    <input
      v-if="mode === 'text' && isAddingText"
      v-model="newText"
      @keyup.enter="confirmText"
      class="text-input"
      :style="{ left: `${textPosition.x}px`, top: `${textPosition.y}px` }"
    />
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
      <text
        v-for="(textElement, index) in texts"
        :key="'text-' + index"
        :x="textElement.x"
        :y="textElement.y"
        :fill="textElement.color"
        :font-size="textElement.fontSize"
        @mouseover="hoverText(index)"
        @mouseout="unhoverText"
        :class="{ highlight: index === hoveredTextIndex }"
      >
        {{ textElement.text }}
      </text>
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

let texts = reactive<
  Array<{ x: number; y: number; text: string; color: string; fontSize: string }>
>([]);
let hoveredTextIndex = ref<number | null>(null);

let isAddingText = ref(false);
let textPosition = ref({ x: 0, y: 0 });
let newText = ref("");

const smoothingFactor = 0.2;

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

const confirmText = () => {
  if (newText.value.trim()) {
    texts.push({
      x: textPosition.value.x,
      y: textPosition.value.y,
      text: newText.value,
      color: selectedColor.value,
      fontSize: selectedStrokeWidth.value + "px",
    });
    newText.value = "";
    isAddingText.value = false;
  }
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
  } else if (mode.value === "text") {
    textPosition.value = { x: e.offsetX, y: e.offsetY };
    isAddingText.value = true;
    return;
  } else if (mode.value === "drawLine") {
    isDrawing = true;
    pathStrings.push(`M${lastX} ${lastY}`);
    strokeWidths.push(selectedStrokeWidth.value);
    colors.push(selectedColor.value);
  } else if (mode.value === "drawArrow") {
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

const clearCanvas = () => {
  pathStrings.length = 0; // Clear all paths
  colors.length = 0; // Clear all colors
  strokeWidths.length = 0; // Clear all stroke widths
  saveSvgData();
};

const undoLastPath = () => {
  if (pathStrings.length > 0) {
    pathStrings.pop();
    colors.pop();
    strokeWidths.pop();
    saveSvgData(); // Update the SVG data after removing the last path
  }
};

const hoverText = (index: number) => {
  if (mode.value === "move") {
    hoveredTextIndex.value = index;
  }
};

const unhoverText = () => {
  hoveredTextIndex.value = null;
};

const handleMouseMove = (e: MouseEvent) => {
  if (mode.value === "draw" && isDrawing && lastX !== null && lastY !== null) {
    let newX = e.offsetX;
    let newY = e.offsetY;

    lastX = lastX + (newX - lastX) * smoothingFactor;
    lastY = lastY + (newY - lastY) * smoothingFactor;

    currentPath.push(`L${lastX.toFixed(2)} ${lastY.toFixed(2)}`);
    pathStrings[pathStrings.length - 1] = currentPath.join(" ");
  } else if (mode.value === "drawArrow" && isDrawing) {
    let endX = e.offsetX;
    let endY = e.offsetY;

    // Создаем стрелку (главная линия плюс две меньшие для кончика стрелки)
    if (lastX && lastY) {
      const arrowString = generateArrowString(lastX, lastY, endX, endY);
      pathStrings[pathStrings.length - 1] = arrowString;
    }
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

const generateArrowString = (
  x1: number,
  y1: number,
  x2: number,
  y2: number
): string => {
  const angle = Math.atan2(y2 - y1, x2 - x1);
  const arrowLength = 30; // можно именить для другого размера стрелки

  const angle1 = angle + (Math.PI * 1) / 6;
  const angle2 = angle - (Math.PI * 1) / 6;

  const x3 = x2 - arrowLength * Math.cos(angle1);
  const y3 = y2 - arrowLength * Math.sin(angle1);

  const x4 = x2 - arrowLength * Math.cos(angle2);
  const y4 = y2 - arrowLength * Math.sin(angle2);

  return `M${x1} ${y1} L${x2} ${y2} M${x2} ${y2} L${x3} ${y3} M${x2} ${y2} L${x4} ${y4}`;
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
.text-input {
  position: absolute;
  background: white;
  border: 1px solid black;
  padding: 5px;
  z-index: 2;
}
</style>
