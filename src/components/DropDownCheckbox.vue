<template>
  <div ref="dropDownRef" class="dropdown-wrapper">
    <button type="button" @click="toggleOptions" class="visible" :aria-expanded="isOpen">
      <div class="selected-options">
        <p v-if="!selectedOptionObjects.length" class="placeholder">Auswählen...</p>
        <div
            v-for="(selected, index) in selectedOptionObjects"
            :key="index"
            class="selected"
            @click.stop="selectOption(selected.value)"
        >
          <p>{{ selected.label }}</p>
          <span class="chip-close" aria-hidden="true">×</span>
        </div>
      </div>
      <span class="chevron" :class="{'rotate': isOpen}" aria-hidden="true">▼</span>
    </button>
    <div v-if="isOpen" class="available-options" role="listbox">
      <button
          type="button"
          :class="{'option': true, 'selected': selectedOptions?.includes(option.value)}"
          v-for="(option, index) in options"
          :key="index"
          @click="selectOption(option.value)"
      >
        {{ option.label }}
      </button>
    </div>
  </div>
</template>

<script setup lang="ts" generic="T">
import {computed, onMounted, onUnmounted, type Ref, ref, useTemplateRef} from "vue";

export interface Option<T> {
  label: string,
  value: T
}

const {options} = defineProps<{
  options: Option<T>[]
}>()

const selectedOptions = defineModel<T[]>('selectedOptions', {
  default: () => [] as T[]
})

const isOpen: Ref<boolean> = ref(false)
const dropDownRef = useTemplateRef<HTMLDivElement>('dropDownRef')

function toggleOptions(): void {
  isOpen.value = !isOpen.value
}

function selectOption(value: T): void {
  const set = new Set(selectedOptions.value)
  set.has(value) ? set.delete(value) : set.add(value)
  selectedOptions.value = [...set]
}

const selectedOptionObjects = computed<Option<T>[]>(() => {
  return options.filter(option =>
      selectedOptions.value.includes(option.value)
  )
})

function handleOutsideClick(event: Event) {
  if (dropDownRef.value && !dropDownRef.value.contains(event.target as Node)) {
    isOpen.value = false
  }
}

onMounted(() => {
  document.addEventListener('click', handleOutsideClick)
})

onUnmounted(() => {
  document.removeEventListener('click', handleOutsideClick)
})
</script>

<style scoped>
.dropdown-wrapper {
  width: 100%;
  position: relative;
  z-index: 5;
}

.visible {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 12px;
  width: 100%;
  min-height: 40px;
  padding: 6px 8px;
  background: #fff;
  border: 2px solid;
  border-color: #404040 #fff #fff #404040;
  cursor: pointer;
  text-align: left;
  color: #000;
  box-shadow: inset 1px 1px 0 #808080;
}

.selected-options {
  display: flex;
  align-items: center;
  flex-wrap: wrap;
  gap: 6px;
  min-width: 0;
  flex: 1;
}

.placeholder {
  color: #808080;
}

.selected {
  display: flex;
  align-items: center;
  gap: 4px;
  font-size: 12px;
  font-weight: 700;
  padding: 2px 6px;
  background: #000080;
  color: #fff;
  border: 1px solid #00f0ff;
}

.chip-close {
  color: #ff2bd6;
  font-size: 14px;
  line-height: 1;
}

.chevron {
  color: #000;
  font-size: 10px;

  &.rotate {
    transform: rotate(180deg);
  }
}

.available-options {
  position: absolute;
  left: 0;
  top: calc(100% + 2px);
  width: 100%;
  background: #fff;
  border: 2px solid;
  border-color: #fff #404040 #404040 #fff;
  box-shadow: 4px 4px 0 #000;
  max-height: min(280px, 50vh);
  overflow: auto;
}

.option {
  display: block;
  width: 100%;
  padding: 8px 10px;
  border: 0;
  background: transparent;
  text-align: left;
  cursor: pointer;
  color: #000;
  font-size: 14px;

  &:hover {
    background: #000080;
    color: #fff;
  }

  &.selected {
    background: #ff2bd6;
    color: #fff;
    font-weight: 700;
  }
}
</style>
