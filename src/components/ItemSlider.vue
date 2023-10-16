<script setup lang="ts">
import type { Item } from '@/types';
import { onMounted, ref } from 'vue';
import gsap from 'gsap';

const props = defineProps<{
    items: Map<number, Item>
}>();

const slideDelay = 1.5;
const slideDuration = 5;
const testIndex = ref(0);
const slideItemsRef = ref<HTMLElement[]>();
const timer = gsap.delayedCall(slideDelay, autoPlay).pause();
const wrapProgress = gsap.utils.wrap(0, 1);
const proxy = document.createElement('div');
const slideWith = ref(0);
const wrapWidth = ref(0);
const wrap = ref<(value: any, target: any) => any>(() => { });
const animation = gsap.timeline({ repeat: -1 });

function autoPlay() {
    animation.play();
    gsap.fromTo(
        animation,
        { timeScale: 0 },
        { timeScale: 1, duration: 1, overwrite: true, ease: 'power1.in' }
    );
};

function resize() {
    const slides = slideItemsRef.value;
    if (!slides) {
        return;
    }
    const numSlides = slides.length;

    const progress = animation.progress();
    slideWith.value = slides[0].offsetWidth;
    wrapWidth.value = slideWith.value * numSlides;

    animation.progress(0)
        .clear()
        .to(slides, {
            duration: 1,
            xPercent: '+=' + numSlides * 100,
            ease: 'none',
            modifiers: {
                xPercent: wrap.value
            }
        })
        .to(proxy, { x: wrapWidth.value, duration: 1, ease: 'none' }, 0)
        .progress(progress);
}

function animateSlides() {
    const slides = slideItemsRef.value;
    if (!slides) {
        return;
    }

    let index = testIndex.value / slides.length;
    index *= -1;
    index += 1;

    timer.pause();
    animation.pause();
    gsap.to(animation, {
        duration: slideDuration,
        progress: index,
        overwrite: true,
        modifiers: {
            progress: wrapProgress // value => (value < 0 || value === 1 ? 1 : 0) + (value % 1)
        },
        onComplete: () => {
            console.log('Complete');
            timer.restart(true);
        }
    });
}

onMounted(() => {
    const slides = slideItemsRef.value;
    if (!slides) {
        return;
    }
    gsap.set(slides, { xPercent: (i) => i * 100 });
    wrap.value = gsap.utils.wrap(-100, (slides.length - 1) * 100);

    resize();
})

</script>

<template>
    <div class="relative flex flex-1 overflow-hidden">
        <ul class="relative h-96 w-96 overflow-hidden">
            <li v-for="[key, item] in items" ref="slideItemsRef"
                class="absolute flex items-center justify-center text-3xl leading-6 font-bold w-full h-full bg-gray-900 p-5 rounded-md">
                {{ key }}
            </li>
        </ul>
    </div>
    <input type="number" class="bg-gray-800" v-model="testIndex">
    <button @click="animateSlides">Stop Slides</button>
</template>