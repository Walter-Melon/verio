<script setup lang="ts">
import type { Item } from '@/types';
import {
  ArrowTrendingUpIcon,
  ArrowTrendingDownIcon,
  TrophyIcon,
  XCircleIcon
} from '@heroicons/vue/24/outline';
import { computed, onMounted, ref } from 'vue';

const props = defineProps<{
  itemKey: number;
  item: Item;
  totalItems: number;
  totalWeight: number;
  disabled?: boolean;
}>();

const emits = defineEmits<{
  (e: 'remove', itemKey: number): void;
  (e: 'mounted', itemKey: number, textField: HTMLInputElement | null): void;
  (e: 'enterPressed', itemKey: number): void;
  (e: 'tabPressed', itemKey: number): void;
}>();

const itemTextRef = ref<HTMLInputElement | null>(null);

const itemChance = computed(() => {
  return (props.item.ignore) ? 0 : parseFloat(((props.item.weight / props.totalWeight) * 100).toFixed(0));
});

const itemHasGoodOdds = computed(() => {
  return itemChance.value >= parseFloat(((1 / props.totalItems) * 100).toFixed(0));
});

const setItemWeight = (weight: number) => {
  props.item.weight = weight;
};

const removeItem = () => {
  emits('remove', props.itemKey);
};

const clickItemNumber = () => {
  props.item.ignore = !props.item.ignore;
}

const handleTabPressed = (event: KeyboardEvent) => {
  // don't emit event on shift + tab press
  if (event.shiftKey) {
    return;
  }

  emits('tabPressed', props.itemKey);
}

const zeroPad = (num: number, places = 2) => String(num).padStart(places, '0');

onMounted(() => {
  emits('mounted', props.itemKey, itemTextRef.value);
});
</script>
<template>
  <li class="flex items-center justify-between border rounded-md gap-x-6 gap-y-4 px-2 py-5">
    <div class="flex flex-wrap items-center gap-x-4 lg:flex-nowrap">
      <div>
        <label :for="`item_${itemKey}`" class="sr-only"> Item {{ itemKey }} </label>
        <div class="flex rounded-md shadow-sm">
          <span :class="[
            (item.ignore) ? 'line-through' : '',
            'inline-flex items-center rounded-l-md cursor-pointer border border-r-0 bg-gray-700 border-gray-700 px-3 text-gray-400 sm:text-sm'
          ]" @click="clickItemNumber">
            {{ zeroPad(itemKey) }}
          </span>
          <input ref="itemTextRef" type="text" v-model="item.text" :name="`item_${itemKey}`" :id="`item_${itemKey}`"
            :disabled="disabled" autofocus :class="[
              (disabled) ? 'opacity-70' : '',
              'block w-full min-w-0 flex-1 rounded-none rounded-r-md border-0 py-1.5 bg-gray-800 text-gray-100 ring-1 ring-inset ring-gray-700 focus:ring-2 focus:ring-inset focus:ring-primary-600 sm:text-sm sm:leading-6'
            ]" @keydown.enter="$emit('enterPressed', itemKey)" @keydown.tab="handleTabPressed" />
        </div>
      </div>
      <div class="mt-1 flex items-center gap-x-2 lg:mt-0">
        <div class="inline-flex items-baseline bg-gray-800 text-gray-400 rounded-full px-2.5 py-0.5 text-sm font-medium">
          <TrophyIcon class="-ml-1 mr-0.5 h-5 w-5 flex-shrink-0 self-center text-gray-500" aria-hidden="true" />
          <span class="sr-only"> Item {{ itemKey }} got choosen </span>
          {{ item.count }}
        </div>
        <div :class="[
          itemHasGoodOdds ? ' text-green-400' : ' text-red-400',
          'inline-flex items-baseline bg-gray-800 rounded-full px-2.5 py-0.5 text-sm font-medium'
        ]">
          <ArrowTrendingUpIcon v-if="itemHasGoodOdds"
            class="-ml-1 mr-0.5 h-5 w-5 flex-shrink-0 self-center text-green-500" aria-hidden="true" />
          <ArrowTrendingDownIcon v-else class="-ml-1 mr-0.5 h-5 w-5 flex-shrink-0 self-center text-red-500"
            aria-hidden="true" />
          <span class="sr-only"> Item {{ itemKey }} odds are </span>
          {{ zeroPad(itemChance) }}%
        </div>
        <div class="inline-flex items-baseline -space-x-0.5" title="Weight">
          <button v-for="i in 5" :key="i" :disabled="disabled" :class="[
            item.weight >= i ? 'bg-primary-500' : 'bg-gray-800',
            'h-6 w-6 rounded-full ring-2 ring-gray-900'
          ]" tabindex="-1" @click="setItemWeight(i)"></button>
        </div>
      </div>
    </div>
    <div class="flex flex-none items-center justify-between gap-x-8 sm:w-auto">
      <button class="bg-gray-900 rounded-full" tabindex="-1" :disabled="disabled" @click="removeItem()">
        <XCircleIcon class="h-6 w-6 text-red-500" aria-hidden="true" />
        <span class="sr-only">Remove Item {{ itemKey }}</span>
      </button>
    </div>
  </li>
</template>
