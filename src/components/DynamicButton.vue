<template>
  <button :class="['button', buttonType]">
    {{ text }}
  </button>
</template>

<script setup lang="ts">
import {ButtonType} from "@/helpers/ButtonType.ts";

const {text, buttonType = ButtonType.Primary} = defineProps<{
  text: string
  buttonType?: ButtonType
}>()

</script>

<style scoped>

.button {
  display: flex;
  justify-content: center;
  align-items: center;
  padding: 4px 12px;
  border-radius: 8px;
  border: 2px solid gray;
  height: 40px;
  cursor: pointer;
  width: fit-content;
  min-width: 200px;

  &:active, &:hover {
    opacity: 0.9;
    transform: scale(1.01);
  }

  &.primary {
    background: lavender;
  }

  &.secondary {
    background: white;
  }

}

</style>