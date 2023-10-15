<template>
  <div>
    <div>
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
        @input="emit('smoothingFactorChange', Number(smoothingFactor))"
      />

      <label for="epsilon">Simplification: {{ epsilon }}</label>
      <input
        type="range"
        id="epsilon"
        v-model="epsilon"
        min="0"
        max="1"
        step="0.1"
        @input="emit('epsilonChange', Number(epsilon))"
      />
      <label for="eraserSize">Eraser Size: {{ eraserSize }}</label>
      <input
        type="range"
        id="eraserSize"
        v-model="eraserSize"
        min="5"
        max="50"
        @input="emit('eraserSizeChange', Number(eraserSize))"
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
        @click="
          activeButton === 'erase'
            ? activateButton('draw')
            : activateButton('erase')
        "
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
const epsilon = ref(0.1);

const eraserSize = ref(25);

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
  "epsilonChange",
  "eraserSizeChange",
]);

onMounted(() => {
  emit("epsilonChange", epsilon.value);
  emit("smoothingFactorChange", smoothingFactor.value);
  emit("colorChange", selectedColor.value);
  emit("eraserSizeChange", eraserSize.value);
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
