<template>
  <button
      type="button"
      :class="['button', buttonType, { block, loading }]"
      :disabled="disabled || loading"
  >
    <span v-if="loading" class="spinner" aria-hidden="true"></span>
    <span>{{ text }}</span>
  </button>
</template>

<script setup lang="ts">
import {ButtonType} from "@/helpers/ButtonType.ts";

const {
  text,
  buttonType = ButtonType.Primary,
  disabled = false,
  loading = false,
  block = false,
} = defineProps<{
  text: string
  buttonType?: ButtonType
  disabled?: boolean
  loading?: boolean
  block?: boolean
}>()

</script>

<style scoped>
.button {
  display: inline-flex;
  justify-content: center;
  align-items: center;
  gap: 8px;
  padding: 8px 16px;
  min-height: 36px;
  cursor: pointer;
  width: fit-content;
  font-family: Tahoma, Verdana, sans-serif;
  font-size: 13px;
  font-weight: 700;
  letter-spacing: 0.04em;
  text-transform: uppercase;
  color: #000;
  background: linear-gradient(#fff, #c0c0c0);
  border: 2px solid;
  border-color: #fff #404040 #404040 #fff;
  box-shadow: 3px 3px 0 #000;

  &:hover:not(:disabled) {
    filter: brightness(1.08);
  }

  &:active:not(:disabled) {
    border-color: #404040 #fff #fff #404040;
    box-shadow: none;
    transform: translate(2px, 2px);
  }

  &:disabled {
    cursor: not-allowed;
    color: #808080;
    filter: grayscale(0.4);
  }

  &.block {
    width: 100%;
  }

  &.primary {
    color: #fff;
    text-shadow: 1px 1px 0 #7a0070;
    background: linear-gradient(#ff9af0, #ff2bd6 45%, #c4009a);
    border-color: #ffc8f6 #7a0070 #7a0070 #ffc8f6;
  }

  &.secondary {
    color: #003;
    background: linear-gradient(#d6fbff, #00d4e6 50%, #0090a8);
    border-color: #fff #006070 #006070 #fff;
  }

  &.ghost {
    background: linear-gradient(#fff, #c0c0c0);
  }
}

.spinner {
  width: 12px;
  height: 12px;
  border: 2px solid rgba(255, 255, 255, 0.35);
  border-top-color: #fff;
  border-radius: 50%;
  animation: spin 700ms linear infinite;
}

@keyframes spin {
  to {
    transform: rotate(360deg);
  }
}
</style>
