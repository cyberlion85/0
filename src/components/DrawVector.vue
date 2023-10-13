<template>
  <div
    :style="{ cursor: cursorComp }"
    id="svg_container"
    style="position: relative"
  >
    <div>
      <svg
        ref="svgRef"
        class="svg-canvas"
        :width="props.videoWidth"
        :height="props.videoHeight"
        @mousedown.left="
          (e) => {
            if (!eraserEnabled) handleMouseDown(e);
          }
        "
        @mousemove="
          (e) => {
            if (!eraserEnabled) handleMouseMove(e);
          }
        "
        @mouseup.left="
          () => {
            if (!eraserEnabled) handleMouseUp();
          }
        "
      >
        <path
          v-for="(pathString, index) in pathStrings"
          :key="index"
          :d="pathString.path"
          :data-type="pathString.type"
          :stroke="pathString.color"
          :stroke-width="pathString.strokeWidth"
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

      <canvas
        ref="canvasRef"
        class="canvas-el"
        :style="{ zIndex: eraserEnabled ? 10 : 0 }"
        :width="props.videoWidth"
        :height="props.videoHeight"
        @mousedown="
          (e) => {
            if (eraserEnabled) handleCanvasMouseDown(e);
          }
        "
        @mousemove="
          (e) => {
            if (eraserEnabled) handleCanvasMouseMove(e);
          }
        "
        @mouseup="
          () => {
            if (eraserEnabled) handleCanvasMouseUp();
          }
        "
      ></canvas>
    </div>
  </div>
</template>

<script setup lang="ts">
import {
  ref,
  withDefaults,
  defineProps,
  defineEmits,
  defineExpose,
  watch,
  onMounted,
  computed,
} from "vue";

const props = withDefaults(
  defineProps<{
    currentFrame: number;
    videoHeight: number;
    videoWidth: number;
    mode: string;
    // isErase: boolean;
    isUndo: boolean;
    selectedColor: string;
    selectedStrokeWidth: string;
    smoothingFactor: number;
    alphaFactor: number;
    epsilon: number;
    eraserSize: number;
  }>(),
  { currentFrame: 0 }
);

interface Point {
  x: number;
  y: number;
}

interface PathInfo {
  type: "draw" | "drawLine" | "drawArrow";
  path: string;
  color: string;
  strokeWidth: number;
}

const emits = defineEmits(["update:framesWithData", "resetUndoClick"]);

// 1. Референсы и контекстыprops.eraserSize
const svgRef = ref<SVGSVGElement | null>(null);
const canvasRef = ref<HTMLCanvasElement | null>(null);
let ctx: CanvasRenderingContext2D | null = null;

// 2. Настройки и состояния инструментов рисования
let isDrawing = false;
let isMoving = false;
let lastX: number | null = null;
let lastY: number | null = null;
const eraserEnabled = ref(false);

// 3. Параметры рисования и ластика
const eraserCursor = ref(""); // курсор ластика

// 4. Данные о путях и координатах
let currentPath: Point[] = [];
let simplifiedPathRes = ref();
let simplifiedLenth = ref(0);
let pathStrings = ref<PathInfo[]>([]);
const svgFramesData: Record<number, PathInfo[]> = {};
const rasterFramesData = ref<Record<number, string>>({});

// 5. Индексы и координаты
let selectedPathIndex = ref<number | null>(null); // Make it reactive
let hoveredPathIndex = ref<number | null>(null);
let initialX = 0;
let initialY = 0;
let deltaX = 0;
let deltaY = 0;

// 6. История и стек
const historyStack = ref<{ canvasData: string; svgData: PathInfo[] }[]>([]);

onMounted(() => {
  if (canvasRef.value) {
    ctx = canvasRef.value.getContext("2d");
  }
  updateEraserCursor();
});

watch(
  () => props.eraserSize,
  () => {
    updateEraserCursor();
  }
);

watch(
  () => props.isUndo,
  (val) => {
    if (val) undo();
    emits("resetUndoClick");
  }
);
watch(
  () => props.selectedStrokeWidth,
  (val) => {
    console.log(val);
  }
);
watch(
  () => props.currentFrame,
  () => {
    loadSvgAndVectorData();
    historyStack.value.length = 0;
  }
);
watch(
  () => props.mode,
  (val) => {
    console.log(val);

    if (val == "erase") {
      eraserEnabled.value = true;
    } else {
      eraserEnabled.value = false;
    }
  }
);

const cursorComp = computed(() => {
  if (eraserEnabled.value) {
    return eraserCursor.value;
  }

  switch (props.mode) {
    case "draw":
      return "crosshair";
    case "move":
      return "move";
    case "delete":
      return "not-allowed";
    default:
      return "default";
  }
});
const updateEraserCursor = () => {
  const svg = `<svg width="${props.eraserSize}" height="${
    props.eraserSize
  }" version="1.1" xmlns="http://www.w3.org/2000/svg">
    <circle cx="${props.eraserSize / 2}" cy="${props.eraserSize / 2}" r="${
    props.eraserSize / 2
  }" fill="rgba(255, 255, 255, 0.5)" stroke="black" stroke-width="1" />
  </svg>`;
  eraserCursor.value = `url("data:image/svg+xml;utf8,${encodeURIComponent(
    svg
  )}") ${props.eraserSize / 2} ${props.eraserSize / 2}, auto`;
};

const undo = () => {
  // console.log("undo clicked");

  if (historyStack.value.length <= 1) {
    return;
  }

  historyStack.value.pop();

  const previousState = historyStack.value[historyStack.value.length - 1];

  pathStrings.value = previousState.svgData;
  rasterFramesData.value[props.currentFrame] = previousState.canvasData;
  svgFramesData[props.currentFrame] = previousState.svgData;

  loadSvgAndVectorData();
};

const rasterize = () => {
  return new Promise<void>((resolve, reject) => {
    const svgElement = svgRef.value;
    const canvasElement = canvasRef.value;
    if (!canvasElement || !svgElement) {
      reject(new Error("SVG or Canvas element is not available"));
      return;
    }

    if (!ctx) {
      reject(new Error("Canvas context is not available"));
      return;
    }

    const paths = svgElement.querySelectorAll(
      "path:not([data-type='drawArrow']):not([data-type='drawLine'])"
    );

    let data = `<svg xmlns="http://www.w3.org/2000/svg" width="${canvasElement.width}" height="${canvasElement.height}">`;

    paths.forEach((path) => {
      data += path.outerHTML;
    });

    data += "</svg>";

    const svg = new Blob([data], { type: "image/svg+xml;charset=utf-8" });
    const url = URL.createObjectURL(svg);

    const img = new Image();

    img.onload = () => {
      ctx?.drawImage(img, 0, 0);
      URL.revokeObjectURL(url);
      resolve();
    };

    img.onerror = () => {
      reject(new Error("Image loading failed"));
    };

    img.src = url;
  });
};

const erase = (x: number, y: number) => {
  const canvasElement = canvasRef.value;
  if (!canvasElement) return;

  if (!ctx) return;

  ctx.globalCompositeOperation = "destination-out";
  ctx.beginPath();
  ctx.arc(x, y, props.eraserSize / 2, 0, Math.PI * 2, true);
  ctx.fill();
  ctx.globalCompositeOperation = "source-over";
};

const handleCanvasMouseDown = (e: MouseEvent) => {
  if (!eraserEnabled.value) return;
  erase(e.offsetX, e.offsetY);
};

const handleCanvasMouseMove = (e: MouseEvent) => {
  if (!eraserEnabled.value) return;

  if (e.buttons !== 1) return;

  erase(e.offsetX, e.offsetY);
};

const handleCanvasMouseUp = () => {
  const canvasElement = canvasRef.value;
  if (!canvasElement || !eraserEnabled.value) return;
  const ctx = canvasElement.getContext("2d");
  if (!ctx) return;
  ctx.globalCompositeOperation = "source-over";
  saveSvgAndVectorData();
};

const saveSvgAndVectorData = () => {
  // console.log("save call");

  // сносим все вевторные рисовалки после растеризации
  // TODO move to rasterize

  pathStrings.value = pathStrings.value.filter((path) => path.type != "draw");
  // console.log(pathStrings.value);

  if (pathStrings.value.length > 0) {
    svgFramesData[props.currentFrame] = [...pathStrings.value];
  }

  if (canvasRef.value) {
    const dataURL = canvasRef.value.toDataURL("image/png");
    rasterFramesData.value[props.currentFrame] = dataURL;
    // console.log(dataURL);
  }

  const framesWithData = Array.from(
    new Set(
      [
        Object.keys(svgFramesData).map(Number),
        ...Object.keys(rasterFramesData.value).map(Number),
      ].flatMap((el) => el)
    )
  );

  addACtionToStack();

  emits("update:framesWithData", framesWithData);
};

const addACtionToStack = () => {
  if (canvasRef.value) {
    historyStack.value.push({
      canvasData: canvasRef.value.toDataURL("image/png"),
      svgData: JSON.parse(JSON.stringify([...pathStrings.value])), // глубокое копирование
    });

    // Длинна стека
    if (historyStack.value.length > 10) {
      historyStack.value.shift(); // Удаление самого старого состояния, если стек превышает 4 элемента
    }
  }
};

const loadSvgAndVectorData = () => {
  pathStrings.value = pathStrings.value.filter((path) => path.type != "draw");
  const svgData = svgFramesData[props.currentFrame];
  if (svgData) {
    pathStrings.value.length = 0;
    svgData.forEach((path) => pathStrings.value.push(path));
    // console.log("Loaded SVG data for frame:", props.currentFrame);
  } else {
    pathStrings.value.length = 0; // Clear the paths if no data for the frame
    // console.log("No SVG data for frame:", props.currentFrame);
  }

  const rasterData = rasterFramesData.value[props.currentFrame];
  const canvasElement = canvasRef.value;
  if (ctx) clearArea(ctx);

  if (rasterData && canvasElement) {
    // console.log(rasterData);

    const ctx = canvasElement.getContext("2d");
    // console.log(rasterFramesData.value);

    const img = new Image();
    img.onload = () => {
      if (ctx) {
        ctx.drawImage(img, 0, 0);
      }
    };
    img.src = rasterData;
  }
};

const clearArea = (ctx: CanvasRenderingContext2D) => {
  if (ctx) {
    ctx.setTransform(1, 0, 0, 1, 0, 0);
    ctx.clearRect(0, 0, ctx.canvas.width, ctx.canvas.height);
  }
};

const handleDeleteClick = () => {
  if (props.mode === "delete") {
    if (hoveredPathIndex.value !== null) {
      pathStrings.value.splice(hoveredPathIndex.value, 1);
      hoveredPathIndex.value = null;
      saveSvgAndVectorData();
    }
  }
};

const handleMouseUp = async () => {
  isDrawing = false;
  isMoving = false;
  lastX = null;
  lastY = null;
  currentPath = [];

  await rasterize();

  saveSvgAndVectorData();

  handleDeleteClick();
};
const handleMouseDown = (e: MouseEvent) => {
  lastX = e.offsetX;
  lastY = e.offsetY;

  if (props.mode === "draw") {
    isDrawing = true;
    pathStrings.value.push({
      type: "draw",
      path: `M${lastX} ${lastY}`,
      color: props.selectedColor,
      strokeWidth: Number(props.selectedStrokeWidth),
    });
    currentPath = [{ x: lastX, y: lastY }];
  } else if (props.mode === "drawLine") {
    isDrawing = true;
    pathStrings.value.push({
      type: "drawLine",
      path: `M${lastX} ${lastY}`,
      color: props.selectedColor,
      strokeWidth: Number(props.selectedStrokeWidth),
    });
  } else if (props.mode === "drawArrow") {
    isDrawing = true;
    pathStrings.value.push({
      type: "drawArrow",
      path: `M${lastX} ${lastY}`,
      color: props.selectedColor,
      strokeWidth: Number(props.selectedStrokeWidth),
    });
  } else if (props.mode === "move") {
    isMoving = true;
    selectedPathIndex.value = hoveredPathIndex.value;
    initialX = e.offsetX;
    initialY = e.offsetY;
  }
};
const handleMouseMove = (e: MouseEvent) => {
  if (props.mode === "draw" && isDrawing && lastX !== null && lastY !== null) {
    const newX = lastX + (e.offsetX - lastX) * props.smoothingFactor;
    const newY = lastY + (e.offsetY - lastY) * props.smoothingFactor;

    lastX = newX;
    lastY = newY;

    currentPath.push({ x: newX, y: newY });

    const currentPoint = { x: e.offsetX, y: e.offsetY };

    const smoothedPAth = smoothPath(
      currentPath,
      props.alphaFactor,
      currentPoint
    ); // Передаем текущую точку чтобы убрать отставание мышки
    const simplifiedPath = ramerDouglasPeucker(smoothedPAth, props.epsilon);
    simplifiedLenth.value = simplifiedPath.length;

    if (simplifiedPath.length) simplifiedPathRes.value = simplifiedPath.length;
    pathStrings.value[pathStrings.value.length - 1].path = simplifiedPath
      .map(
        ({ x, y }, idx) =>
          `${idx === 0 ? "M" : "L"}${x.toFixed(2)} ${y.toFixed(2)}`
      )
      .join(" ");
  } else if (props.mode === "drawArrow" && isDrawing) {
    let endX = e.offsetX;
    let endY = e.offsetY;

    // Создаем стрелку (главная линия плюс две меньшие для кончика стрелки)
    if (lastX && lastY) {
      const arrowString = generateArrowString(lastX, lastY, endX, endY);
      pathStrings.value[pathStrings.value.length - 1].path = arrowString;
    }
  } else if (props.mode === "drawLine" && isDrawing) {
    let endX = e.offsetX;
    let endY = e.offsetY;
    pathStrings.value[
      pathStrings.value.length - 1
    ].path = `M${lastX} ${lastY} L${endX} ${endY}`;
  } else if (
    props.mode === "move" &&
    isMoving &&
    selectedPathIndex.value !== null
  ) {
    // console.log(props.mode);

    deltaX = e.offsetX - initialX;
    deltaY = e.offsetY - initialY;
    initialX = e.offsetX;
    initialY = e.offsetY;

    let movedPath = pathStrings.value[selectedPathIndex.value].path.replace(
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
    pathStrings.value[selectedPathIndex.value].path = movedPath;
  }
};

const hoverPath = (index: number) => {
  if (props.mode === "move" || props.mode === "delete") {
    hoveredPathIndex.value = index;
  }
};

const clearScreen = () => {
  pathStrings.value.length = 0; // Clear all paths
  if (ctx) clearArea(ctx);
  saveSvgAndVectorData();
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

defineExpose({
  saveSvgAndVectorData,
  loadSvgAndVectorData,
  clearScreen,
});
</script>

<style scoped>
.svg-canvas {
  /* border: 5px solid red; */
  opacity: 0.4;
  position: absolute;
  z-index: 1;
}
.canvas-el {
  /* border: 5px dotted blue; */
  position: absolute;
  /* z-index: 0; */
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
