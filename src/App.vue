<script setup lang="ts">
import { PicsumImage } from "./interfaces/picsumimage.interface.ts";

import ImageCarousel from "./components/ImageCarousel.vue";
import { computed, onMounted, ref } from "vue";

const listImages = ref<PicsumImage[]>([]);
// const selectedImages = [];

const getImages = async () => {
  try {
    return await fetch("https://picsum.photos/v2/list").then(
      (response) => response.json() as Promise<PicsumImage[]>
    );
  } catch (err) {
    console.error(err);
    return [];
  }
};

const updateSelectValue = (id: string) => {
  const listIndex = listImages.value.findIndex((image) => image.id == id);
  if (listIndex >= 0) {
    listImages.value[listIndex].selected =
      !listImages.value[listIndex].selected;
  }
};

const selectedImages = computed(() => {
  return listImages.value.filter((image) => image.selected);
});

onMounted(async () => {
  listImages.value = await getImages();
});
</script>

<template>
  <ImageCarousel :list-image="listImages" @toggle-select="updateSelectValue" />

  <transition-group
    tag="ul"
    name="selected-images-list"
    class="selected-images-list"
  >
    <li
      class="list-item"
      v-for="image in selectedImages"
      :key="image.id"
      v-if="selectedImages.length > 0"
    >
      <p
        class="author"
        @click="updateSelectValue(image.id)"
        tabindex="0"
        @keydown.enter="updateSelectValue(image.id)"
      >
        {{ image.author }}
      </p>
      <span class="image-id">{{ image.id }}</span>
      <a :href="image.url" target="_blank" class="image_url">{{
        image.download_url
      }}</a>
    </li>
    <li class="list-item" v-else>
      <p>No images selected yet</p>
    </li>
  </transition-group>
</template>

<style scoped lang="scss">
.selected-images-list {
  margin-top: 50px;
  list-style: none;
  display: grid;
  gap: 0.7rem;
  justify-content: center;
  .list-item {
    display: grid;
    grid-template-columns: 1fr auto;
    gap: 1rem;
    color: white;
    background: #2d2d2d;
    padding: 0.7rem 2rem;
    transition: all 0.3s linear;
    justify-items: start;
    user-select: none;

    .author {
      white-space: nowrap;
      transition: color 0.3s linear;
      cursor: pointer;
      &:hover {
        color: #d5d5d5;
      }
    }
    .image-url {
      grid-column: span 2;
    }
  }
}

.selected-images-list-move,
.selected-images-list-enter-active,
.selected-images-list-leave-active {
  transition: all 1s ease;
}

.selected-images-list-enter-from,
.selected-images-list-leave-to {
  opacity: 0;
  transform: translateX(-50px);
}

.selected-images-list-leave-active {
  position: absolute;
}
</style>
