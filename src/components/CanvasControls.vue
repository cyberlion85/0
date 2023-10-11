<template>
  <div>
    <div style="margin-top: 10px">
      <label for="smoothingFactor"
        >Smoothing Factor: {{ smoothingFactor }}</label
      >
      <input
        type="range"
        id="smoothingFactor"
        v-model="smoothingFactor"
        min="0.1"
        max="1"
        step="0.1"
        @input="emit('smoothingFactorChange', smoothingFactor)"
      />

      <label for="alphaFactor">Alpha Factor Func: {{ alphaFactor }}</label>
      <input
        type="range"
        id="alphaFactor"
        v-model="alphaFactor"
        min="0.1"
        max="1"
        step="0.1"
        @input="emit('alphaFactorChange', alphaFactor)"
      />

      <label for="epsilon">Epsilon: {{ epsilon }}</label>
      <input
        type="range"
        id="epsilon"
        v-model="epsilon"
        min="0"
        max="1"
        step="0.1"
        @input="emit('epsilonChange', epsilon)"
      />
    </div>
    <div class="control-buttons">
      <button @click="emit('draw')">Draw</button>
      <button @click="emit('drawLine')">Line</button>
      <button @click="emit('drawArrow')">Arrow</button>
      <button @click="emit('erase')">Erase</button>
      <button @click="emit('move')">Move</button>
      <!-- переделать на define expose -->
      <button @click="emit('undo')">Undo</button>
      <button @click="emit('clear')">Clear</button>
      <button @click="emit('delete')">Delete Vector</button>

      <input
        type="color"
        v-model="selectedColor"
        @change="emit('colorChange', selectedColor)"
      />
      <select
        @change="emit('strokeWidthChange', selectedStrokeWidth)"
        v-model="selectedStrokeWidth"
      >
        <option value="1">1px</option>
        <option value="3">3px</option>
        <option value="5">5px</option>
        <option value="7">7px</option>
        <option value="10">10px</option>
      </select>
      <slot></slot>
      <!-- Слот для дополнительных элементов управления -->
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, defineProps, defineEmits, onMounted } from "vue";

// Состояние цвета
const selectedColor = ref("#f6b73c");
const selectedStrokeWidth = ref("3");

// Параметры сглаживания и упрощения
const smoothingFactor = ref(0.5);
const alphaFactor = ref(0.5);
const epsilon = ref(0.1);

// Определение входных параметров (если нужны)
defineProps([]);

// Определение событий, которые этот компонент может вызывать
const emit = defineEmits([
  "draw",
  "drawArrow",
  "drawLine",
  "erase",
  "delete",
  "move",
  "undo",
  "clear",
  "colorChange",
  "strokeWidthChange",
  "smoothingFactorChange",
  "alphaFactorChange",
  "epsilonChange",
]);

onMounted(() => {
  emit("epsilonChange", epsilon.value);
  emit("smoothingFactorChange", smoothingFactor.value);
  emit("alphaFactorChange", alphaFactor.value);
  emit("colorChange", selectedColor.value);
});
</script>

<style scoped>
.control-buttons button {
  margin: 5px;
  padding: 10px;
  font-size: 16px;
  cursor: pointer;
}
</style>
