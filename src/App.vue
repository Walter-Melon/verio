<script setup lang="ts">
import type { Item } from './types';
import { computed, reactive, ref } from 'vue';
import gsap from 'gsap';
import ItemInputList from './components/ItemInputList.vue';

const items = ref<Map<number, Item>>(new Map());

const choosenKey = ref<number>();

const tweened = reactive({
  turn: 0
});

const totalWeight = computed(() => {
  return [...items.value.values()].reduce((acc, items) => acc + items.weight, 0);
});

// current turn gets wrapped around to an index
const actIndex = computed(() => {
  return parseInt((tweened.turn % items.value.size).toString());
});

// Total cyles + offset for index to get correct item
const totalTurns = computed(() => {
  return cyles * items.value.size + (choosenKey.value || 0 % items.value.size);
});

const cyles = 20;

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

const startAnimation = () => {
  const key = getRandomItemKey();
  if (!key) {
    return;
  }

  tweened.turn = 0;

  gsap.to(tweened, {
    duration: 10,
    ease: 'power1.inOut',
    turn: totalTurns.value,
    onComplete: () => {
      choosenKey.value = key;
    }
  });
}

</script>

<template>
  <div class="flex flex-col gap-2 p-2">
    <ItemInputList class="w-full" :items="items" :total-weight="totalWeight" :choosen-key="choosenKey"
      :active-index="actIndex" />
    <button class="w-full h-full flex-1 bg-primary-500 rounded-md text-gray-800 font-bold p-2"
      @click="startAnimation">Decide</button>
  </div>
</template>
