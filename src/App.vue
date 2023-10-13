<script setup lang="ts">
import { computed, ref } from 'vue';
import type { Item } from './types';
import { PlusCircleIcon, TrashIcon } from '@heroicons/vue/20/solid';
import ItemInputGroup from './components/ItemInputGroup.vue';

const nextItemKey = ref(1);
const items = ref<Map<number, Item>>(new Map());
const choosenKey = ref<number>();
const choosenItem = ref<Item>();
const startCount = ref(3);
const focusKey = ref<number>(0);

const totalWeight = computed(() => {
  return [...items.value.values()].reduce((acc, items) => acc + items.weight, 0);
});

const addItem = ({ text, setFocus = false }: { text?: string, setFocus?: boolean } = {}): number => {
  const itemKey = nextItemKey.value;
  nextItemKey.value++;

  items.value.set(itemKey, {
    text: text ?? `Item ${itemKey}`,
    weight: 1,
    count: 0
  });

  if (setFocus) {
    focusKey.value = itemKey;
  }

  return itemKey;
};

const getItem = (key: number) => {
  const item = items.value.get(key);
  if (!item) {
    throw new Error(`No item with key "${key}" found`);
  }

  return item;
};

const removeItem = (key: number) => {
  items.value.delete(key);

  if (items.value.size <= 0) {
    nextItemKey.value = 1;
  }
};

const chooseRandomItem = () => {
  const key = getRandomItemKey();
  if (!key) {
    return;
  }

  const item = getItem(key);

  item.count++;
  choosenItem.value = item;
  choosenKey.value = key;
};

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

const addItems = () => {
  for (let i = 1; i <= startCount.value; i++) {
    addItem({
      setFocus: (i == 1)
    });
  }
};

const removeItems = () => {
  items.value.clear();
  nextItemKey.value = 1;
};

const itemInputMounted = (key: number, inputField: HTMLInputElement | null) => {
  if (key == focusKey.value) {
    inputField?.select();
    focusKey.value = 0;
  }
}

const testChooseItem = (times: number) => {
  items.value.forEach((item, key) => item.count = 0);

  for (let i = 0; i < times; i++) {
    chooseRandomItem();
  }
};
</script>

<template>
  <header class="flex gap-2 p-2">
    <div class="flex-grow">
      <label for="item_count" class="sr-only">Add items</label>
      <div class="flex rounded-md shadow-sm">
        <div class="relative flex items-stretch focus-within:z-10">
          <input type="number" v-model="startCount" autofocus
            class="block w-full rounded-none rounded-l-md border-0 py-1.5 bg-gray-900 text-gray-100 ring-1 ring-inset ring-gray-800 focus:ring-2 focus:ring-inset focus:ring-primary-600 sm:text-sm sm:leading-6"
            @keyup.enter="addItems()" />
        </div>
        <button type="button"
          class="flex-grow relative -ml-px inline-flex items-center justify-center gap-x-1.5 rounded-r-md px-3 py-2 text-sm font-semibold bg-gray-900 text-primary-600 ring-1 ring-inset ring-gray-800 hover:bg-gray-700"
          @click="addItems()">
          <PlusCircleIcon class="-ml-0.5 h-5 w-5 text-primary-500" aria-hidden="true" />
          Add Items
        </button>
      </div>
    </div>
    <button type="button"
      class="flex items-center justify-center gap-x-1.5 rounded-md p-2 text-sm font-semibold bg-gray-900 text-red-600 ring-1 ring-inset ring-gray-800 hover:bg-gray-700"
      @click="removeItems()">
      <TrashIcon class="h-5 w-5 text-red-500" aria-hidden="true" />
    </button>
  </header>

  <main class="p-2">
    <div class="flex flex-col gap-2 ">
      <ul class="flex flex-col gap-1">
        <ItemInputGroup v-for="[key, item] in items" :key="key" :index="key" :total-items="items.size"
          :total-weight="totalWeight" :choosen-key="choosenKey" :item="item" @remove="removeItem"
          @mounted="itemInputMounted" />
      </ul>
      <button type="button"
        class="flex w-full items-center justify-center gap-x-1.5 rounded-md p-2 text-sm font-semibold bg-gray-900 text-primary-600 ring-1 ring-inset ring-gray-800 hover:bg-gray-700"
        @click="addItem({ setFocus: true })">
        <PlusCircleIcon class="h-5 w-5 text-primary-500" aria-hidden="true" />
        Add Item
      </button>
    </div>
    <div class="flex gap-2">
      <button @click="chooseRandomItem()">Get Item</button>
      <button @click="testChooseItem(1000)">Test 1000</button>
    </div>
  </main>
</template>
