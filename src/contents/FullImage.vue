<template>
    <div class="zoom" @touchstart="touchstart" @touchmove="touchmove" @touchend="touchend"
        :style="`transform: translate3d(${style.x}px, ${style.y}px, 0) scale(${style.scale})`">
        <img ref="image" class="position-relative z-1" :src="image_options.preview" loading="lazy">
    </div>
</template>

<script setup>
import { ref } from 'vue';

const emit = defineEmits(['open_post']);
const props = defineProps({
    data: {
        type: Object,
        required: true
    }
})

const style = ref({ x: 0, y: 0, scale: 1 });
const start = ref({ x: 0, y: 0, distance: 0 });

const image = ref(null);
const image_options = ref({});

const distance = (event) => {
    return Math.hypot(event.touches[0].clientX - event.touches[1].clientX, event.touches[0].clientY - event.touches[1].clientY);
};

const midpoint_x = (event) => {
    return (event.touches[0].clientX + event.touches[1].clientX) / 2;
};

const midpoint_y = (event) => {
    return (event.touches[0].clientY + event.touches[1].clientY) / 2;
};

async function touchstart(event) {
    if (event.touches.length === 2) {
        event.preventDefault();

        start.value.x = midpoint_x(event);
        start.value.y = midpoint_y(event);
        start.value.distance = distance(event);
    }
}

async function touchmove(event) {
    if (event.targetTouches.length === 2) {
        event.preventDefault();

        let scale = distance(event) / start.value.distance;

        style.value.scale = scale;
        style.value.x = start.value.x - (start.value.x * scale) + (midpoint_x(event) - start.value.x);
        style.value.y = start.value.y - (start.value.y * scale) + (midpoint_y(event) - start.value.y);
    }
}

async function touchend() {
    // Reset image to it's original format
    style.value = { x: 0, y: 0, scale: 1 };
}

async function get_sources() {
    if (!props.data.preview) {
        image_options.value = {
            src: props.data.url,
            preview: props.data.url,
            style: {
                'aspect-ratio': `${props.data.thumbnail.width} / ${props.data.thumbnail.height}}`
            }
        }
        return;
    }

    image_options.value = {
        src: props.data.url,
        preview: props.data.preview.images[0].resolutions.pop().url.replaceAll("&amp;", "&"),
        style: {
            'aspect-ratio': `${props.data.preview.images[0].resolutions.slice(-1)[0].width} / ${props.data.preview.images[0].resolutions.slice(-1)[0].height}`
        }
    }
}

// setup
get_sources();
</script>