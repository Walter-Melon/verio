<script setup lang="ts">
import { computed, ref } from 'vue';
import type { Item } from './types';
import ItemInputList from './components/ItemInputList.vue';
import ItemSlider from './components/ItemSlider.vue';

const items = ref<Map<number, Item>>(new Map());

const totalWeight = computed(() => {
  return [...items.value.values()].reduce((acc, items) => acc + items.weight, 0);
});

</script>

<template>
  <div class="grid grid-cols-1 gap-2 p-2 sm:grid-cols-2">
    <ItemInputList class="w-full hidden" :items="items" :total-weight="totalWeight" />
    <ItemSlider :items="items" />
  </div>
</template>
