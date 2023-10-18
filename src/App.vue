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

const decideBtnText = computed(() => {
  return (deciding.active) ? 'Skip' : 'Decide';
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

const decide = () => {
  // Skip the animation
  if (deciding.active) {
    finishedDeciding();
    return;
  }

  choosenKey.value = undefined;
  deciding.active = true;
  deciding.turn = 0;
  deciding.key = getRandomItemKey();

  if (!deciding.key) {
    return;
  }

  tween.value = gsap.to(deciding, {
    duration: 10,
    ease: 'power1.inOut',
    turn: totalTurns.value, // This animates the deciding.turn value
    onComplete: finishedDeciding
  });
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
        :current-index="currentIndex" :deciding="deciding.active" @item-enter-pressed="decide" />
      <div class="sticky py-2 bg-gray-950 bottom-0">
        <button class="w-full bg-primary-500 rounded-md text-gray-800 font-bold p-2" @click="decide">
          {{ decideBtnText }}
        </button>
      </div>
    </div>
  </div>
</template>
