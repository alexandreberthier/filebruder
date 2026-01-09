<template>
  <div ref="dropDownRef" class="dropdown-wrapper">
    <div @click="toggleOptions" class="visible">
      <div class="selected-options">
        <p v-if="!selectedOptionObjects.length" class="">Mandant auswählen</p>
        <div @click.stop="selectOption(selected.value)" v-for="(selected, index) in selectedOptionObjects" :key="index"
             class="selected">
          <p>{{ selected.label }}</p>
          <img :src="getImage('ic_close.png')" alt="">
        </div>
      </div>
      <div class="icon-wrapper">
        <img :class="{'rotate': isOpen}" :src="getImage('ic_chevron.png')" alt="">
      </div>
    </div>
    <div v-if="isOpen" class="available-options">
      <p :class="{'selected':selectedOptions?.includes(option.value) }"
         @click="selectOption(option.value)"
         v-for="(option, index) in options"
         :key="index">{{ option.label }}
      </p>
    </div>
  </div>
</template>

<script setup lang="ts" generic="T">
import {computed, type ModelRef, onMounted, onUnmounted, type Ref, ref, useTemplateRef} from "vue";
import {getImage} from "@/helpers/ImageUtils.ts";

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
  display: flex;
  flex-direction: column;
  gap: 4px;
  width: fit-content;
  min-width: 300px;
  cursor: pointer;
  position: relative;
  background: white;
  z-index: 5;

  .visible {
    display: flex;
    align-items: center;
    justify-content: space-between;
    border-radius: 8px;
    border: 1px solid gray;
    padding: 4px 12px;
    gap: 32px;
    height: 50px;
    background: white;

    .selected-options {
      display: flex;
      align-items: center;
      gap: 8px;

      .selected {
        background: lavender;
        padding: 4px 8px;
        border-radius: 8px;
        display: flex;
        align-items: center;
        gap: 4px;

        img {
          width: 14px;
          height: 14px;

        }
      }
    }

    .icon-wrapper {
      display: flex;
      justify-content: center;
      align-items: center;

      img {
        width: 24px;
        height: 24px;
        transition: all 250ms ease-in-out;

        &.rotate {
          transform: rotate(180deg);
        }
      }
    }
  }

  .available-options {
    display: flex;
    flex-direction: column;
    gap: 4px;
    border: 1px solid gray;
    padding: 4px 12px;
    border-radius: 8px;
    position: absolute;
    left: 0;
    top: 105%;
    background: white;
    width: 100%;


    p {
      padding: 8px;
      border-radius: 8px;

      &.selected {
        background: lightgrey;
      }

      &:hover {
        background: lavender;
      }
    }
  }
}

</style>