<template>
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
    <slot></slot>
    <!-- Слот для дополнительных элементов управления -->
  </div>
</template>

<script setup lang="ts">
import { ref, defineProps, defineEmits } from "vue";

// Состояние цвета
const selectedColor = ref("#000000");

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
]);
</script>

<style scoped>
/* Стили для кнопок (при необходимости можно доработать) */
.control-buttons button {
  margin: 5px;
  padding: 10px;
  font-size: 16px;
  cursor: pointer;
}
</style>
