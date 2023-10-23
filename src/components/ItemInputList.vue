<script setup lang="ts">
import type { Item } from '@/types';
import { ref, watch } from 'vue';
import { PlusCircleIcon, TrashIcon } from '@heroicons/vue/20/solid';
import ItemInputGroup from './ItemInputGroup.vue';


const props = defineProps<{
    items: Map<number, Item>;
    totalWeight: number;
    choosenKey?: number;
    currentKey?: number;
    deciding?: boolean;
}>();

const emits = defineEmits<{
    (e: 'itemEnterPressed', key: number): void;
}>();

const nextItemKey = ref(1);
const startCount = ref(3);
const focusKey = ref<number>(0);
const inputGroups = ref<InstanceType<typeof ItemInputGroup>[]>([]);

const addItem = ({ text, setFocus = false }: { text?: string, setFocus?: boolean } = {}): number => {
    const itemKey = nextItemKey.value;
    nextItemKey.value++;

    props.items.set(itemKey, {
        text: text ?? `Item ${itemKey}`,
        weight: 3,
        count: 0
    });

    if (setFocus) {
        focusKey.value = itemKey;
    }

    return itemKey;
};

const getItem = (key: number) => {
    const item = props.items.get(key);
    if (!item) {
        throw new Error(`No item with key "${key}" found`);
    }

    return item;
};

const removeItem = (key: number) => {
    props.items.delete(key);

    if (props.items.size <= 0) {
        nextItemKey.value = 1;
    }
};

const addItems = () => {
    for (let i = 1; i <= startCount.value; i++) {
        addItem({
            setFocus: (i == 1)
        });
    }
};

const removeItems = () => {
    props.items.clear();
    nextItemKey.value = 1;
};

const itemInputMounted = (key: number, inputField: HTMLInputElement | null) => {
    if (key == focusKey.value) {
        inputField?.select();
        focusKey.value = 0;
    }
}

const itemEnterPressed = (key: number) => {
    emits('itemEnterPressed', key);
}

watch(() => props.choosenKey, (key, oldKey) => {
    if (!key) {
        return;
    }

    const item = getItem(key);
    item.count++;

    if (inputGroups.value.length <= 0) {
        return;
    }

    inputGroups.value.forEach((itemGroup) => {
        if (itemGroup.$props.itemKey != key) {
            return;
        }

        const el: HTMLElement = itemGroup.$el;
        el.scrollIntoView({ behavior: 'smooth' });
    });
});

addItems();
</script>

<template>
    <div class="flex flex-col gap-2">
        <header :class="[
            (deciding) ? 'opacity-70' : '',
            'flex gap-2'
        ]">
            <div class="flex-grow">
                <label for="item_count" class="sr-only">Add items</label>
                <div class="flex rounded-md shadow-sm">
                    <div class="relative flex items-stretch focus-within:z-10">
                        <input type="number" v-model="startCount" :disabled="deciding" autofocus
                            class="block w-full rounded-none rounded-l-md border-0 py-1.5 bg-gray-900 text-gray-100 ring-1 ring-inset ring-gray-800 focus:ring-2 focus:ring-inset focus:ring-primary-600 sm:text-sm sm:leading-6"
                            @keyup.enter="addItems()" />
                    </div>
                    <button type="button" :disabled="deciding"
                        class="flex-grow relative -ml-px inline-flex items-center justify-center gap-x-1.5 rounded-r-md px-3 py-2 text-sm font-semibold bg-gray-900 text-primary-600 ring-1 ring-inset ring-gray-800 hover:bg-gray-700 disabled:bg-gray-900"
                        @click="addItems()">
                        <PlusCircleIcon class="-ml-0.5 h-5 w-5 text-primary-500" aria-hidden="true" />
                        Add Items
                    </button>
                </div>
            </div>
            <button type="button" :disabled="deciding"
                class="flex items-center justify-center gap-x-1.5 rounded-md p-2 text-sm font-semibold bg-gray-900 text-red-600 ring-1 ring-inset ring-gray-800 hover:bg-gray-700 disabled:bg-gray-900"
                @click="removeItems()">
                <TrashIcon class="h-5 w-5 text-red-500" aria-hidden="true" />
            </button>
        </header>

        <main>
            <div class="flex flex-col gap-2 ">
                <ul class="flex flex-col gap-1">
                    <ItemInputGroup ref="inputGroups" v-for="[key, item] in items" :key="key" :item-key="key"
                        :disabled="deciding" :total-items="items.size" :total-weight="totalWeight" :item="item" :class="[
                            (key == currentKey || key == choosenKey) ? 'border-primary-600' : 'border-gray-800',
                            (key == choosenKey && !item.ignore) ? 'bg-primary-600/30' : 'bg-gray-900',
                            (deciding) ? 'opacity-70' : '',
                        ]" @remove="removeItem" @mounted="itemInputMounted" @enter-pressed="itemEnterPressed" />
                </ul>
                <button type="button" :class="[
                    (deciding) ? 'opacity-70' : '',
                    'flex w-full items-center justify-center gap-x-1.5 rounded-md p-2 text-sm font-semibold bg-gray-900 text-primary-600 ring-1 ring-inset ring-gray-800 hover:bg-gray-700 disabled:bg-gray-900'
                ]" :disabled="deciding" @click="addItem({ setFocus: true })">
                    <PlusCircleIcon class="h-5 w-5 text-primary-500" aria-hidden="true" />
                    Add Item
                </button>
            </div>
        </main>
    </div>
</template>