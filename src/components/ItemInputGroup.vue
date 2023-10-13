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
  index: number;
  item: Item;
  totalItems: number;
  totalWeight: number;
  choosenKey?: number;
}>();

const emits = defineEmits<{
  (e: 'remove', index: number): void;
  (e: 'mounted', index: number, textField: HTMLInputElement | null): void;
}>();

const itemTextRef = ref<HTMLInputElement | null>(null);

const itemChance = computed(() => {
  return parseFloat(((props.item.weight / props.totalWeight) * 100).toFixed(2));
});

const itemHasGoodOdds = computed(() => {
  return itemChance.value >= parseFloat(((1 / props.totalItems) * 100).toFixed(2));
});

const setItemWeight = (weight: number) => {
  props.item.weight = weight;
};

const removeItem = () => {
  emits('remove', props.index);
};

onMounted(() => {
  emits('mounted', props.index, itemTextRef.value);
});
</script>
<template>
  <li :class="[
    index == choosenKey ? 'border-primary-600' : 'border-gray-800',
    'flex flex-wrap items-center justify-between bg-gray-900 border rounded-md gap-x-6 gap-y-4 px-2 py-5 sm:flex-nowrap'
  ]">
    <div>
      <div>
        <label :for="`item_${index}`" class="sr-only"> Item {{ index }} </label>
        <div class="mt-2 flex rounded-md shadow-sm">
          <span
            class="inline-flex items-center rounded-l-md border border-r-0 bg-gray-700 border-gray-700 px-3 text-gray-400 sm:text-sm">
            {{ index }}
          </span>
          <input ref="itemTextRef" type="text" v-model="item.text" :name="`item_${index}`" :id="`item_${index}`" autofocus
            class="block w-full min-w-0 flex-1 rounded-none rounded-r-md border-0 py-1.5 bg-gray-800 text-gray-100 ring-1 ring-inset ring-gray-700 focus:ring-2 focus:ring-inset focus:ring-primary-600 sm:text-sm sm:leading-6" />
        </div>
      </div>
      <div class="mt-1 flex items-center gap-x-2">
        <div class="flex w-16 gap-x-2.5">
          <div
            class="inline-flex items-baseline bg-gray-400/10 text-gray-400 rounded-full px-2.5 py-0.5 text-sm font-medium md:mt-2 lg:mt-0">
            <TrophyIcon class="-ml-1 mr-0.5 h-5 w-5 flex-shrink-0 self-center text-gray-500" aria-hidden="true" />
            <span class="sr-only"> Item {{ index }} got choosen </span>
            {{ item.count }}
          </div>
          <div :class="[
            itemHasGoodOdds ? 'bg-green-400/10 text-green-400' : 'bg-red-400/10 text-red-400',
            'inline-flex items-baseline rounded-full px-2.5 py-0.5 text-sm font-medium md:mt-2 lg:mt-0'
          ]">
            <ArrowTrendingUpIcon v-if="itemHasGoodOdds"
              class="-ml-1 mr-0.5 h-5 w-5 flex-shrink-0 self-center text-green-500" aria-hidden="true" />
            <ArrowTrendingDownIcon v-else class="-ml-1 mr-0.5 h-5 w-5 flex-shrink-0 self-center text-red-500"
              aria-hidden="true" />
            <span class="sr-only"> Item {{ index }} odds are </span>
            {{ itemChance }}
          </div>
        </div>
      </div>
    </div>
    <dl class="flex w-full flex-none items-center justify-between gap-x-8 sm:w-auto">
      <div class="flex -space-x-0.5" title="Weight">
        <dt class="sr-only">Weight</dt>
        <dd v-for="i in 5" :key="i">
          <button :class="[
            item.weight >= i ? 'bg-primary-500' : 'bg-gray-800',
            'h-6 w-6 rounded-full ring-2 ring-gray-900'
          ]" tabindex="-1" @click="setItemWeight(i)"></button>
        </dd>
      </div>
      <div class="">
        <dt class="sr-only">Remove Item {{ index }}</dt>
        <dd>
          <button class="bg-gray-900 rounded-full" tabindex="-1" @click="removeItem()">
            <XCircleIcon class="h-6 w-6 text-red-500" aria-hidden="true" />
          </button>
        </dd>
      </div>
    </dl>
  </li>
</template>
