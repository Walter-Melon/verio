<script setup lang="ts">
import { ref } from 'vue';


interface Item {
  label: string;
  weight: number;
  choosen: number;
}

const choose = ref<Item>();
const total = ref(0);

const items = ref<Item[]>([
  { label: 'Test 1', weight: 0.8, choosen: 0 },
  { label: 'Test 2', weight: 0.2, choosen: 0 },
]);

const testRandom = () => {
  total.value = 0;
  for (const item of items.value) {
    item.choosen = 0;
  }

  for (let i = 0; i < 1000; i++) {
    choose.value = getRandomItem();
  }
}

const getItemOdds = (item: Item): number => {
  const totalWeight = items.value.reduce((acc, items) => acc + items.weight, 0);
  return parseFloat(((item.weight / totalWeight) * 100).toFixed(2));
}

const getRandomItem = (): Item | undefined => {
  const totalWeight = items.value.reduce((acc, items) => acc + items.weight, 0);
  let randomValue = Math.random() * totalWeight;

  total.value++;

  for (const item of items.value) {
    randomValue -= item.weight;
    if (randomValue <= 0) {
      item.choosen++;
      return item;
    }
  }

  return;
}

</script>

<template>
  <header>
    <div class="container">
      <h2 class="text-2xl text-primary-500">Test</h2>
      <span>Total: {{ total }}</span>
    </div>
  </header>

  <main>
    <ul>
      <li v-for="(item) in items">
        {{ item.label }} ({{ getItemOdds(item) }}%)
        <input type="number" v-model="item.weight" class="bg-gray-900">
        <span>Choosen: {{ item.choosen }}</span>
      </li>
    </ul>

    <div class="flex gap-2">
      <button @click="() => choose = getRandomItem()">Start</button>
      <button @click="testRandom()">Test</button>
      <span>Choose: </span>
      <span>{{ choose?.label }}</span>
    </div>

  </main>
</template>