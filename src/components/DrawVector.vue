<template>
  <div id="svg_container">
    <button @click="(mode = 'draw'), (selectedStrokeWidth = 5)">Draw</button>
    <button @click="mode = 'move'">Move</button>
    <button @click="mode = 'delete'">Delete Mode</button>
    <button @click="(mode = 'drawLine'), (selectedStrokeWidth = 3)">
      Draw Line
    </button>
    <button @click="(mode = 'drawArrow'), (selectedStrokeWidth = 3)">
      Draw Arrow
    </button>
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
        :class="[
          index === hoveredPathIndex
            ? mode === 'move'
              ? 'highlight'
              : mode === 'delete'
              ? 'highlight-delete'
              : ''
            : '',
        ]"
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

interface Point {
  x: number;
  y: number;
}

const emits = defineEmits(["update:frameToImageData"]);

let mode = ref("draw");
let isDrawing = false;
let isMoving = false;
let currentPath: Point[] = [];
let simplifiedPathRes = ref();
let pathStrings: string[] = reactive([]);
let selectedPathIndex = ref<number | null>(null); // Make it reactive
let hoveredPathIndex = ref<number | null>(null);
let lastX: number | null = null;
let lastY: number | null = null;
let initialX = 0;
let initialY = 0;
let deltaX = 0;
let deltaY = 0;
const epsilon = 0.1;
const smoothingFactor = 1; //1 выкл

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

const handleDeleteClick = () => {
  if (mode.value === "delete") {
    if (hoveredPathIndex.value !== null) {
      pathStrings.splice(hoveredPathIndex.value, 1);
      colors.splice(hoveredPathIndex.value, 1);
      strokeWidths.splice(hoveredPathIndex.value, 1);
      hoveredPathIndex.value = null;
      saveSvgData();
    }
  }
};

const handleMouseUp = () => {
  isDrawing = false;
  isMoving = false;
  lastX = null;
  lastY = null;
  currentPath = [];
  saveSvgData();

  handleDeleteClick();
};

// Обновлённый обработчик наведения на объект
const hoverPath = (index: number) => {
  if (mode.value === "move" || mode.value === "delete") {
    hoveredPathIndex.value = index;
  }
};

const handleMouseDown = (e: MouseEvent) => {
  lastX = e.offsetX;
  lastY = e.offsetY;

  if (mode.value === "draw") {
    isDrawing = true;
    currentPath = [{ x: lastX, y: lastY }];
    pathStrings.push("");
    strokeWidths.push(selectedStrokeWidth.value);
    colors.push(selectedColor.value);
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

const handleMouseMove = (e: MouseEvent) => {
  if (mode.value === "draw" && isDrawing && lastX !== null && lastY !== null) {
    console.log(isDrawing);
    console.log(lastX);
    console.log(lastY);

    const newX = lastX + (e.offsetX - lastX) * smoothingFactor;
    const newY = lastY + (e.offsetY - lastY) * smoothingFactor;

    lastX = newX;
    lastY = newY;

    currentPath.push({ x: newX, y: newY });
    const currentPoint = { x: e.offsetX, y: e.offsetY };

    const smoothedPAth = smoothPath(currentPath, 0.1, currentPoint); // Передаем текущую точку чтобы убрать отставание мышки
    const simplifiedPath = ramerDouglasPeucker(smoothedPAth, epsilon);

    if (simplifiedPath.length) simplifiedPathRes.value = simplifiedPath.length;
    pathStrings[pathStrings.length - 1] = simplifiedPath
      .map(
        ({ x, y }, idx) =>
          `${idx === 0 ? "M" : "L"}${x.toFixed(2)} ${y.toFixed(2)}`
      )
      .join(" ");
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

  // Добавляем текущую точку курсора в конец сглаженного пути
  smoothed.push(currentPoint);

  return smoothed;
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
.highlight-delete {
  stroke: red;
  cursor: pointer;
}
</style>
