<template>
  <div class="slidecontainer relative flex">
    <div class="pointer-events-none absolute flex w-full justify-center">
      <div
        ref="progress"
        class="fillLeft h-1 bg-primary"
        :style="{ width: progressWidth, margin: marginProgress }"
      ></div>
    </div>
    <input
      type="range"
      :min="min"
      :max="max"
      :step="step"
      v-model="sliderValue"
      class="slider"
      id="myRange"
      ref="range"
      @input="$emit('update:modelValue', Number(($event.target as HTMLInputElement).value))"
    />
  </div>
</template>
<script setup lang="ts">
import { computed } from 'vue';
import { ref } from 'vue';

interface ISliderProps {
  modelValue: number;
  min: number;
  max: number;
  step: number;
}

const props = defineProps<ISliderProps>();

const progress = ref<HTMLDivElement>();
const range = ref();

const progressWidth = computed(
  () =>
    `${Math.abs((sliderValue.value / (props.max / 100) / 100) * (range.value?.clientWidth / 2))}px`,
);
const marginProgress = computed(() =>
  sliderValue.value >= 0
    ? `0 0 0 ${Math.abs(
        (sliderValue.value / (Math.abs(props.min) / 100) / 100) * (range.value?.clientWidth / 2),
      )}px`
    : `0 ${Math.abs(
        (sliderValue.value / (Math.abs(props.max) / 100) / 100) * (range.value?.clientWidth / 2),
      )}px 0 0`,
);

const sliderValue = ref(props.modelValue);
</script>
<style lang="scss" scoped>
.slider {
  appearance: none;
  width: 100%;
  height: 4px;
  border-radius: 5px;
  background: #eaedf1;
  outline: none;
  -webkit-transition: 0.2s;
  transition: opacity 0.2s;
}

.slider::-webkit-slider-thumb {
  -webkit-appearance: none;
  appearance: none;
  width: 14px;
  height: 14px;
  border-radius: 50%;
  background: #3b82f6;
  cursor: pointer;
}

.slider::-moz-range-thumb {
  width: 25px;
  height: 25px;
  border-radius: 50%;
  background: #3b82f6;
  cursor: pointer;
}
</style>
