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
      <button
        @click="activateButton('draw')"
        :class="{ active: activeButton === 'draw' }"
      >
        Draw
      </button>
      <button
        @click="activateButton('drawLine')"
        :class="{ active: activeButton === 'drawLine' }"
      >
        Line
      </button>
      <button
        @click="activateButton('drawArrow')"
        :class="{ active: activeButton === 'drawArrow' }"
      >
        Arrow
      </button>
      <button
        @click="activateButton('erase')"
        :class="{ active: activeButton === 'erase' }"
      >
        Erase
      </button>
      <button
        @click="activateButton('move')"
        :class="{ active: activeButton === 'move' }"
      >
        Move
      </button>
      <button
        @click="activateButton('delete')"
        :class="{ active: activeButton === 'delete' }"
      >
        Delete Vector
      </button>
      <!-- переделать на define expose -->
      <button @click="emit('undo')">Undo</button>
      <button @click="emit('clear')">Clear</button>

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
import { ref, defineEmits, onMounted } from "vue";

// Состояние цвета
const selectedColor = ref("#f6b73c");
const selectedStrokeWidth = ref("3");
const activeButton = ref("");

// Параметры сглаживания и упрощения
const smoothingFactor = ref(0.5);
const alphaFactor = ref(0.5);
const epsilon = ref(0.1);

const activateButton = (
  buttonName: "draw" | "drawLine" | "drawArrow" | "erase" | "move" | "delete"
) => {
  activeButton.value = buttonName;
  emit(buttonName);
};

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

.control-buttons button.active {
  background-color: #f0ad4e;
  color: white;
  border-color: #eea236;
}
</style>
