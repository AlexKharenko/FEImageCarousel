<script setup lang="ts">
import { onMounted, ref, watch } from "vue";

import { PicsumImage } from "../interfaces/picsumimage.interface";

type Props = {
  listImage: PicsumImage[];
};

const props = defineProps<Props>();
const emit = defineEmits(["toggleSelect"]);
const visibleImages = ref<PicsumImage[]>([]);

const activeCardWidth = 300;

const currentImageId = ref();
let middleImageIndex = ref(0);

const nextImage = () => {
  const currentImageIndex = visibleImages.value.findIndex(
    (image) => image.id == currentImageId.value
  );
  currentImageId.value = visibleImages.value[currentImageIndex + 1].id;
  visibleImages.value.shift();
  const newElement = getLastNextImage(visibleImages.value.slice(-1)[0].id);
  visibleImages.value.push(newElement);
};

const prevImage = () => {
  const currentImageIndex = visibleImages.value.findIndex(
    (image) => image.id == currentImageId.value
  );
  currentImageId.value = visibleImages.value[currentImageIndex - 1].id;
  visibleImages.value.pop();
  const newElement = getLastPrevImage(visibleImages.value[0].id);
  visibleImages.value.unshift(newElement);
};

const getLastNextImage = (id: string): PicsumImage => {
  const currentImageIndex = props.listImage.findIndex(
    (image) => image.id == id
  );
  const lastNextImageIndex = (currentImageIndex + 1) % props.listImage.length;
  return props.listImage[lastNextImageIndex];
};

const getLastPrevImage = (id: string): PicsumImage => {
  const currentImageIndex = props.listImage.findIndex(
    (image) => image.id == id
  );
  const lastPrevImageIndex =
    currentImageIndex - 1 >= 0
      ? currentImageIndex - 1
      : props.listImage.length - 1;
  return props.listImage[lastPrevImageIndex];
};

const initVisibleImages = (array: PicsumImage[]) => {
  if (array.length >= 7) {
    visibleImages.value = array.slice(0, 7);
  } else if (array.length > 0 && array.length < 7) {
    visibleImages.value = array;
  }
  if (visibleImages.value.length > 0) {
    middleImageIndex.value = Math.floor(visibleImages.value.length / 2);
    currentImageId.value = visibleImages.value[middleImageIndex.value].id;
  }
};

const handleImageClick = (id: string) => {
  if (currentImageId.value == id) emit("toggleSelect", id);
};

watch(
  () => props.listImage,
  (newPropValue) => {
    if (newPropValue) {
      initVisibleImages(newPropValue);
    }
  }
);

onMounted(() => {
  initVisibleImages(props.listImage);
});
</script>

<template>
  <div class="image-carousel-container">
    <div
      class="image-carousel"
      :style="{
        translate: `calc(-200px - ${
          middleImageIndex * activeCardWidth
        }px - ${middleImageIndex} * 1rem)`,
      }"
    >
      <div
        class="card"
        v-if="visibleImages.length > 0"
        v-for="image in visibleImages"
        :key="image.id"
        :class="currentImageId == image.id ? 'active' : ''"
        @click="handleImageClick(image.id)"
        @keydown.enter="handleImageClick(image.id)"
        :tabindex="currentImageId == image.id ? '1  ' : ''"
      >
        <img
          class="card-image"
          :src="image.download_url"
          :alt="`Image Author ${image.author}`"
          loading="lazy"
        />
        <transition name="selected-lable">
          <div class="selected-lable" v-if="image.selected">Selected</div>
        </transition>
        <div class="card-text-content">
          <h3 class="author">{{ image.author }} {{ image.id }}</h3>
        </div>
      </div>
    </div>
    <div class="btn-control-block prev" v-if="listImage.length > 2">
      <button class="btn-control" @click="prevImage" tabindex="0">
        <img src="../assets/right-arrow.png" alt="Prev image" />
      </button>
    </div>
    <div class="btn-control-block next" v-if="listImage.length > 2">
      <button class="btn-control" @click="nextImage" tabindex="0">
        <img src="../assets/right-arrow.png" alt="Prev image" />
      </button>
    </div>
  </div>
</template>

<style scoped lang="scss">
.image-carousel-container {
  position: relative;
  width: 100%;
  margin: 0 auto;
  max-width: 1280px;
  height: 600px;
  display: flex;
  align-items: center;
  overflow: hidden;
  .btn-control-block {
    position: absolute;
    display: flex;
    height: 450px;
    padding: 0.6rem;
    flex-direction: column;
    justify-content: center;
    background: linear-gradient(to right, rgba(78, 76, 76, 0), black);

    &.next {
      right: 0;
    }
    &.prev {
      left: 0;
      transform: rotate(180deg);
    }
    .btn-control {
      height: 40px;
      width: 40px;
      outline: none;
      background: none;
      border: none;
      display: flex;
      justify-content: center;
      align-items: center;
      cursor: pointer;
      &:hover img,
      &:focus-visible img {
        filter: invert(0.8);
      }
      img {
        transition: filter 0.2s linear;
        filter: invert(1);
      }
    }
  }
  .image-carousel {
    position: absolute;
    display: flex;
    left: 50%;
    align-items: center;
    gap: 1rem;
    overflow: hidden;
    user-select: none;
    .card {
      position: relative;
      height: 450px;
      width: 300px;
      min-width: 300px;
      overflow: hidden;
      transition: height 0.3s linear, width 0.3s linear;

      &.active {
        height: 600px;
        width: 400px;
        min-width: 400px;
        cursor: pointer;
      }
      .card-image {
        height: inherit;
        width: inherit;
        object-fit: cover;
        transition: scale 0.2s linear;
      }
      .selected-lable {
        display: flex;
        align-items: center;
        gap: 0.4rem;
        padding: 0.5rem 0.7rem;
        position: absolute;
        top: 10px;
        right: 20px;
        background: #2d2d2d;
        color: #eae6e6;
        border-radius: 5px;
        letter-spacing: 0.14em;
        font-weight: 500;
        &::before {
          content: "";
          height: 10px;
          width: 10px;
          background: #eae6e6;
          border-radius: 50%;
        }
      }
      .card-text-content {
        position: absolute;
        bottom: 0;
        left: 0;
        display: flex;
        align-items: flex-end;
        padding: 1rem;
        width: inherit;
        height: 200px;
        color: rgb(220, 218, 218);
        background: linear-gradient(#25252500, #252525);
      }
      &:hover:is(.active) {
        .card-image {
          scale: 1.2;
        }
      }
    }
  }
}

.selected-lable-enter-active,
.selected-lable-leave-active {
  transition: opacity 0.3s ease;
}

.selected-lable-enter-from,
.selected-lable-leave-to {
  opacity: 0;
}

@media screen and (max-width: 480px) {
  .image-carousel-container {
    margin: 0;
    .btn-control-block {
      height: 100%;
      background: none;
      .btn-control {
        background: rgba(58, 57, 57, 0.587);
        border-radius: 3px;
      }
    }
    .image-carousel {
      translate: -50% !important;
      .card {
        width: 0;
        min-width: auto;
        &.active {
          width: 100vw;
          min-width: auto;
        }
      }
      .card-text-content {
        white-space: nowrap;
      }
    }
  }
}
</style>
