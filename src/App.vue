<script setup lang="ts">
import type { Item } from './types';
import { computed, reactive, ref } from 'vue';
import gsap from 'gsap';
import ItemInputList from './components/ItemInputList.vue';

const items = ref<Map<number, Item>>(new Map());
const choosenKey = ref<number>();

const cyles = 20;
const tween = ref<GSAPTween>();

const deciding = reactive<{
  active: boolean;
  turn: number;
  key?: number;
}>({
  active: false,
  turn: 0,
});

const totalWeight = computed(() => {
  return [...items.value.values()].reduce((acc, items) => acc + items.weight, 0);
});

// current turn gets wrapped around to an index
const currentIndex = computed(() => {
  if (!deciding.active) {
    return;
  }

  return Math.floor(deciding.turn % items.value.size);
});

// Total cyles + offset for index to get correct item
const totalTurns = computed(() => {
  return cyles * items.value.size + (deciding.key || 0 % items.value.size);
});

const getRandomItemKey = (): number | undefined => {
  let randomValue = Math.random() * totalWeight.value;

  for (const [key, item] of items.value) {
    randomValue -= item.weight;
    if (randomValue <= 0) {
      return key;
    }
  }

  return;
};

const decide = (skipAnimation = false) => {
  // Skip the animation
  if (deciding.active) {
    finishedDeciding();
    return;
  }

  startDeciding();

  if (!deciding.key) {
    return;
  }

  if (skipAnimation) {
    finishedDeciding();
    return;
  }

  tween.value = gsap.to(deciding, {
    duration: 10,
    ease: 'power1.inOut',
    turn: totalTurns.value, // This animates the deciding.turn value
    onComplete: finishedDeciding
  });
}

const startDeciding = () => {
  choosenKey.value = undefined;
  deciding.active = true;
  deciding.turn = 0;
  deciding.key = getRandomItemKey();
}

const finishedDeciding = () => {
  tween.value?.kill();

  choosenKey.value = deciding.key;
  deciding.active = false;
}

</script>

<template>
  <div class="mx-auto max-w-7xl pb-0 p-2 sm:p-6 lg:p-8">
    <div class="flex flex-col gap-2 mx-auto max-w-3xl">
      <ItemInputList class="w-full" :items="items" :total-weight="totalWeight" :choosen-key="choosenKey"
        :current-index="currentIndex" :deciding="deciding.active" @item-enter-pressed="decide()" />
      <div class="sticky py-2 bg-gray-950 bottom-0">
        <span v-if="!deciding.active" class="isolate w-full inline-flex rounded-md shadow-sm">
          <button type="button"
            class="relative w-full inline-flex items-center justify-center rounded-l-md bg-primary-500 p-2 font-bold text-gray-900 hover:bg-primary-700 focus:z-10"
            @click="decide()">
            Decide
          </button>
          <button type="button"
            class="relative w-full -ml-px inline-flex items-center justify-center rounded-r-md bg-gray-900 p-2 font-semibold text-primary-500 hover:bg-gray-700 focus:z-10">
            Chicken Dinner
          </button>
        </span>
        <button v-else type="button"
          class="relative w-full inline-flex items-center justify-center rounded-md bg-primary-500 p-2 font-bold text-gray-900 hover:bg-primary-700 focus:z-10"
          @click="decide(true)">
          Skip...
        </button>
      </div>
    </div>
  </div>
</template>
