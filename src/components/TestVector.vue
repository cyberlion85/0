<template>
  <div>
    <canvas
      id="drawingCanvas"
      :height="canvasHeight"
      :width="canvasWidth"
      ref="canvasRef"
      style="border: 1px solid black"
    ></canvas>
    <div>{{ textContent }}</div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted } from "vue";
import paper from "paper";

const canvasRef = ref<HTMLCanvasElement | null>(null);
const textContent = ref<string>("Click and drag to draw a line.");
const canvasHeight = 500;
const canvasWidth = 500;
const strokeWidth = 2;

onMounted(() => {
  paper.install(window);
  if (canvasRef.value) {
    paper.setup(canvasRef.value);
  }

  var path: paper.Path;
  var textItem = new paper.PointText(new paper.Point(20, 30));
  textItem.fillColor = new paper.Color(0.5, 0, 0.5);
  textItem.content = textContent.value;

  const tool = new paper.Tool();
  tool.minDistance = 10; // Set the min distance between points

  tool.onMouseDown = (t) => {
    if (path) {
      path.selected = false;
    }

    path = new paper.Path();
    path.strokeColor = new paper.Color(0.5, 0, 0.5);
    path.strokeWidth = strokeWidth;
    path.fullySelected = true;
  };

  tool.onMouseDrag = (event: paper.ToolEvent) => {
    path.add(event.point);
    textContent.value = "Segment count: " + path.segments.length;
  };

  tool.onMouseUp = () => {
    const segmentCount = path.segments.length;
    path.smooth({ type: "continuous", factor: 1 });
    path.simplify();
    path.fullySelected = false;

    const newSegmentCount = path.segments.length;
    const difference = segmentCount - newSegmentCount;
    const percentage = 100 - Math.round((newSegmentCount / segmentCount) * 100);
    textContent.value =
      difference +
      " of the " +
      segmentCount +
      " segments were removed. Saving " +
      percentage +
      "%";
  };
});
</script>

<style scoped>
#drawingCanvas {
  border: 1px solid black;
}
</style>
