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

const filteredItems = computed(() => {
  return new Map([...items.value].filter(([key, item]) => !item.ignore));
});

const totalWeight = computed(() => {
  return [...filteredItems.value.values()].reduce((acc, items) => acc + items.weight, 0);
});

// current turn gets wrapped around to an index
// and fetches the key belonging to the index
const currentKey = computed(() => {
  if (!deciding.active) {
    return;
  }

  const index = Math.floor(deciding.turn % filteredItems.value.size);

  return [...filteredItems.value.keys()][index];
});

// Total cyles + offset for index to get correct item
const totalTurns = computed(() => {
  return cyles * filteredItems.value.size + (deciding.key || 0 % filteredItems.value.size);
});

const getRandomItemKey = (): number | undefined => {
  let randomValue = Math.random() * totalWeight.value;

  for (const [key, item] of filteredItems.value) {
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

const excludeDecide = () => {
  // Decide as long as more then one filtered item exists
  if (filteredItems.value.size <= 1) {
    startDeciding();
    finishedDeciding();
    choosenKey.value = [...filteredItems.value.keys()][0];
    return;
  }

  startDeciding();

  tween.value = gsap.to(deciding, {
    duration: 2,
    ease: 'power1.inOut',
    turn: totalTurns.value,
    onComplete: () => {
      finishedDeciding();
      if (!choosenKey.value) {
        return;
      }

      const item = filteredItems.value.get(choosenKey.value);
      if (!item) {
        return;
      }

      item.ignore = true;

      excludeDecide();
    }
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
        :current-key="currentKey" :deciding="deciding.active" @item-enter-pressed="decide()" />
      <div class="sticky py-2 bg-gray-950 bottom-0">
        <span v-if="!deciding.active" class="isolate w-full inline-flex rounded-md shadow-sm">
          <button type="button"
            class="relative w-full inline-flex items-center justify-center rounded-l-md bg-primary-500 p-2 font-bold text-gray-900 hover:bg-primary-700 focus:z-10"
            @click="decide()">
            Decide
          </button>
          <button type="button"
            class="relative w-full -ml-px inline-flex items-center justify-center rounded-r-md bg-gray-900 p-2 font-semibold text-primary-500 hover:bg-gray-700 focus:z-10"
            @click="excludeDecide()">
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
