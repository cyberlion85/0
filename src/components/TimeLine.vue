<template>
  <div class="timeline-container">
    <!-- Кнопка Play -->
    <div class="play-button">Play</div>
    <div class="timeline">
      <div
        v-for="frame in totalFrames"
        :key="frame"
        class="frame"
        :class="[
          frame % 2 === 0 ? 'blue-frame' : 'grey-frame',
          frame === selectedFrame ? 'selected-frame' : '',
          frame === playingFrame ? 'current-frame' : '',
          props.framesWithSketch.includes(frame) ? 'sketch-frame' : '',
        ]"
        @mouseover="
          (evt) => {
            highlightFrame(frame), playFramesByMouse(evt, frame);
          }
        "
        @mouseout="unhighlightFrame"
        @click="selectFrame(frame)"
      >
        <div v-if="frame === selectedFrame" class="frame-number">
          {{ frame }}
        </div>
      </div>
    </div>
    <div class="frame-number-container">Frame: {{ highlightedFrame }}</div>
  </div>
</template>

<script setup lang="ts">
import { ref, withDefaults, defineProps, defineEmits } from "vue";

const props = withDefaults(
  defineProps<{
    totalFrames: number;
    playingFrame: number;
    framesWithSketch: number[];
  }>(),
  {}
);

const emits = defineEmits(["selectedFrame"]);

const highlightedFrame = ref(0);
const selectedFrame = ref(0);

const highlightFrame = (frame: number) => {
  highlightedFrame.value = frame;
};
const playFramesByMouse = (evt: MouseEvent, frame: number) => {
  if (evt.buttons === 1) {
    emits("selectedFrame", frame);
  }
};

const unhighlightFrame = () => {
  highlightedFrame.value = 0;
};

const selectFrame = (frame: number) => {
  selectedFrame.value = frame;
  emits("selectedFrame", selectedFrame.value);
};
</script>

<style scoped>
.timeline-container {
  display: grid;
  grid-template-columns: 70px 1fr 70px;
  align-items: center;
  width: 100%;
  box-sizing: border-box;
}

.play-button,
.frame-number-container {
  display: flex;
  justify-content: center;
  align-items: center;
  border-right: 2px solid rgb(64, 239, 6);
  height: 35px;
  min-width: 70px;
  background-color: rgb(65, 65, 65);
  color: white;
  font-size: 16px;
}

.timeline {
  display: flex;
  height: 35px;
}

.play-button:hover {
  background-color: rgb(92, 92, 92);
  cursor: pointer;
}

.frame {
  /* flex: 1; */
  min-width: 2px;
  width: 100%;
  position: relative;
}

.blue-frame {
  background-color: rgb(61, 61, 61);
}

.grey-frame {
  background-color: rgb(65, 65, 65);
}

.selected-frame {
  background-color: rgb(205, 255, 203);
}

.current-frame {
  background-color: rgb(255, 255, 0);
}
.sketch-frame {
  background-color: rgb(10, 250, 6);
}

.frame:hover {
  background-color: rgb(205, 255, 203);
}

.frame-number {
  position: absolute;
  top: -20px;
  left: 50%;
  transform: translateX(-50%);
  color: rgb(3, 128, 1);
}
</style>
