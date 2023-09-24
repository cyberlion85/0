<template>
  <!-- props.framesWithSketch.includes(frame) ? 'sketch-frame' : '', -->
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
      @mouseover="highlightFrame(frame)"
      @mouseout="unhighlightFrame"
      @click="selectFrame(frame)"
    >
      <div v-if="frame === selectedFrame" class="frame-number">
        {{ frame }}
      </div>
    </div>
  </div>
  {{ highlightedFrame }}
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

const unhighlightFrame = () => {
  highlightedFrame.value = 0;
};

const selectFrame = (frame: number) => {
  selectedFrame.value = frame;
  emits("selectedFrame", selectedFrame.value);
};
</script>

<style scoped>
.timeline {
  display: flex;
  flex-direction: row;
  width: 100%;
}

.frame {
  flex: 1;
  height: 50px;
  position: relative;
}

.blue-frame {
  background-color: rgb(81, 81, 81);
}

.grey-frame {
  background-color: rgb(55, 55, 55);
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
