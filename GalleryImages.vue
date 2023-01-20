<script lang="ts" setup>
import {
    reactive,
    computed,
    watch,
    onBeforeUnmount,
    ref,
} from 'vue';
import {
    Textarea as ATextarea,
} from 'ant-design-vue';
import PhotoSwipe from 'photoswipe';
import PhotoSwipeLightbox from 'photoswipe/lightbox';
import { CollectPhoto } from '@/typings/Requests';
import { DocModel } from '@/typings/Docs';

import STUB from '@/assets/img/noPhoto.svg';

import 'photoswipe/dist/photoswipe.css';

const props = defineProps<{
    photo: Array<string>,
    doc: DocModel,
    cmtPlaceholder: string,
    width?: number,
    height?: number,
}>();

const state = reactive({
    fullWidth:  0,
    fullHeight: 0,
    isReady:    false,
    isError:    false,
    isComment:  false,
});

const text = ref<string>('');

const isStub = computed(() => state.isError || !props.photo);

let lightbox: {
    init(): void,
    destroy(): void,
    on(name: string, fn: any):void,
    loadAndOpen(index: number): void,
} | undefined;

/**
 * Формируем нужный формат с данными
 *
 * @returns {Array} Возвращаем массив изображений
 */
const collectDataPhoto = (): CollectPhoto[] => props.photo.map(el => ({
    src: el || STUB,
    w:   state.fullWidth,
    h:   state.fullHeight,
}));

/**
 * Инициализация галлереии
 */
function initializeLightbox() {
    lightbox = new PhotoSwipeLightbox({
        dataSource:         collectDataPhoto(),
        arrowPrev:          true,
        arrowNext:          true,
        counter:            false,
        wheelToZoom:        true,
        initialZoomLevel:   'fit',
        secondaryZoomLevel: 2,
        maxZoomLevel:       4,
        pswpModule:         PhotoSwipe,
    });

    if (lightbox) {
        lightbox.on('uiRegister', () => {
            lightbox.ui.registerElement({
                // undefined
            })
        });

        lightbox.init();
    }
}

/**
 * Обработка клика на открытие полноэкранного просмотра
 *
 * @type {number} index Индекс элемента на который нажали
 */
function handleClick(index: number) {
    if (!lightbox) return;

    lightbox.loadAndOpen(index);
    state.isComment = true;
}

/**
 * Сброс состояния
 */
function resetState() {
    state.isError = false;
    state.isReady = false;
    state.fullHeight = 0;
    state.fullWidth = 0;
    state.isComment = false;

    if (lightbox != null) {
        lightbox.destroy();
        lightbox = undefined;
    }
}

/**
 * Обработка ошибки загрузки
 */
function handingError() {
    state.isError = true;
}

/**
 * Обработка загрузки фото
 *
 * @type {Event} event Принимаем изображение
 */
function photoProcessingOnLoad(event: Event) {
    if (!isStub.value) {
        const img = event.target as HTMLImageElement;

        state.fullWidth = img.naturalWidth;
        state.fullHeight = img.naturalHeight;
    }

    state.isReady = true;
}

watch(() => props.photo, () => resetState());
watch(() => state.isReady, (isReady) => {
    if (!isReady || isStub.value) return;

    initializeLightbox();
});

onBeforeUnmount(() => {
    if (!lightbox) return;

    lightbox.destroy();
    lightbox = undefined;
});
</script>

<template>
    <div class="gallery">
        <figure
            v-for="(item, index) in photo"
            :key="item"
            class="gallery__figure"
            @click="handleClick(index)"
        >
            <img
                :src="item"
                alt="Фото галлереии"
                :height="height"
                :width="width"
                :class="{
                    'gallery__figure-img': true,
                    'gallery__figure-img--cover': isStub,
                }"
                loading="lazy"
                @load="photoProcessingOnLoad"
                @error="handingError"
            >
        </figure>

        <div class="gallery__comments">
            <ATextarea
                v-if="state.isComment"
                :rows="4"
                v-model:value="text"
                :placeholder="cmtPlaceholder"
                class="gallery__comments-inp"
            />
        </div>
    </div>
</template>

<style lang="less">
.gallery {
    display: flex;
    flex-wrap: wrap;

    &__figure-img {
        width: 416px;
        height: 350px;
        margin: 12px 12px 0 0;
        border-radius: 8px;
        object-fit: cover;
    }

    &__comments {
        position: fixed;
        top: 0;
        left: 0;
        z-index: 2;
    }
}

.pswp {
    z-index: 1;
}
</style>
