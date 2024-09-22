<template>
    <div class="image-viewer">
      <div v-if="isInitialLoading" class="loading">
        <div class="spinner"></div>
        Loading images...
      </div>
      <div v-else class="image-container" @mousemove="handleMouseMove" @mouseleave="handleMouseLeave">
        <transition name="fade-slide" mode="out-in">
          <div v-if="isImageLoading" key="loading" class="loading-overlay">
            <div class="spinner"></div>
          </div>
          <img
            v-else
            :key="currentIndex"
            :src="currentImage"
            :alt="`Image ${currentIndex + 1}`"
            class="viewer-image"
            @load="handleImageLoad"
          />
        </transition>
        <div class="image-zoom-container" :style="zoomContainerStyle">
          <img
            :src="currentImage"
            :alt="`Image ${currentIndex + 1} (zoomed)`"
            class="zoom-image"
          />
        </div>
        <div class="image-counter">{{ currentIndex + 1 }} / {{ images.length }}</div>
      </div>
      <div class="controls">
        <button @click="prevImage" :disabled="currentIndex === 0" class="nav-button">
          <ChevronLeftIcon />
        </button>
        <button @click="nextImage" :disabled="currentIndex === images.length - 1" class="nav-button">
          <ChevronRightIcon />
        </button>
      </div>
    </div>
  </template>
  
  <script setup>
  import { ref, computed, watch } from 'vue'
  import { ChevronLeftIcon, ChevronRightIcon } from 'lucide-vue-next'
  
  const props = defineProps({
    images: {
      type: Array,
      required: true
    }
  })
  
  const currentIndex = ref(0)
  const scale = ref(1)
  const translateX = ref(0)
  const translateY = ref(0)
  const isInitialLoading = ref(true)
  const isImageLoading = ref(true)
  const isHovering = ref(false)
  
  const currentImage = computed(() => props.images[currentIndex.value])
  
  const zoomContainerStyle = computed(() => ({
    transform: `scale(${scale.value})`,
    transformOrigin: `${translateX.value}% ${translateY.value}%`,
    opacity: isHovering.value ? 1 : 0,
  }))
  
  const nextImage = () => {
    if (currentIndex.value < props.images.length - 1) {
      currentIndex.value++
    }
  }
  
  const prevImage = () => {
    if (currentIndex.value > 0) {
      currentIndex.value--
    }
  }
  
  const handleMouseMove = (event) => {
    const { left, top, width, height } = event.currentTarget.getBoundingClientRect()
    const x = ((event.clientX - left) / width) * 100
    const y = ((event.clientY - top) / height) * 100

    scale.value = 1.2
    translateX.value = x
    translateY.value = y
    isHovering.value = true
  }
  
  const handleMouseLeave = () => {
    scale.value = 1
    translateX.value = 50
    translateY.value = 50
    isHovering.value = false
  }
  
  const handleImageLoad = () => {
    isImageLoading.value = false
    isInitialLoading.value = false
  }
  
  const loadImage = (src) => {
    isImageLoading.value = true
    const img = new Image()
    img.onload = handleImageLoad
    img.src = src
  }
  
  watch(() => props.images, (newImages) => {
    if (newImages.length > 0) {
      isInitialLoading.value = true
      loadImage(newImages[0])
    }
  }, { immediate: true })
  
  watch(currentIndex, (newIndex) => {
    loadImage(props.images[newIndex])
  })
  </script>
  
  <style scoped>
  .image-viewer {
    max-width: 800px;
    margin: 0 auto;
    background-color: #1a1a1a;
    border-radius: 12px;
    overflow: hidden;
    box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1), 0 1px 3px rgba(0, 0, 0, 0.08);
  }
  
  .image-container {
    position: relative;
    aspect-ratio: 16 / 9;
    overflow: hidden;
    min-height: 450px; /* Add this line to set a minimum height */
  }
  
  .viewer-image {
    width: 100%;
    height: 100%;
    object-fit: cover;
    display: block;
  }
  
  .image-zoom-container {
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    overflow: hidden;
    pointer-events: none;
    transition: transform 0.1s ease-out, opacity 0.2s ease-out;
  }
  
  .zoom-image {
    width: 100%;
    height: 100%;
    object-fit: cover;
  }
  
  .image-counter {
    position: absolute;
    bottom: 16px;
    right: 16px;
    background-color: rgba(0, 0, 0, 0.6);
    color: white;
    padding: 4px 8px;
    border-radius: 12px;
    font-size: 14px;
  }
  
  .controls {
    display: flex;
    justify-content: space-between;
    padding: 16px;
  }
  
  .nav-button {
    background-color: rgba(255, 255, 255, 0.1);
    border: none;
    color: white;
    padding: 8px;
    border-radius: 50%;
    cursor: pointer;
    transition: background-color 0.3s ease, transform 0.3s ease;
  }
  
  .nav-button:hover:not(:disabled) {
    background-color: rgba(255, 255, 255, 0.2);
    transform: scale(1.1);
  }
  
  .nav-button:disabled {
    opacity: 0.5;
    cursor: not-allowed;
  }
  
  .fade-slide-enter-active,
  .fade-slide-leave-active {
    transition: all 0.5s ease-out;
  }
  
  .fade-slide-enter-from {
    opacity: 0;
    transform: translateX(50px);
  }
  
  .fade-slide-leave-to {
    opacity: 0;
    transform: translateX(-50px);
  }
  
  @media (max-width: 768px) {
    .image-viewer {
      border-radius: 0;
    }
  
    .image-container {
      aspect-ratio: 4 / 3;
    }
  
    .image-counter {
      bottom: 8px;
      right: 8px;
      font-size: 12px;
    }
  
    .controls {
      padding: 8px;
    }
  }
  
  .loading, .loading-overlay {
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    height: 450px;
    color: white;
    font-size: 1.2em;
    background-color: rgba(0, 0, 0, 0.7);
  }
  
  .loading-overlay {
    position: absolute;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
  }
  
  .spinner {
    width: 50px;
    height: 50px;
    border: 3px solid rgba(255, 255, 255, 0.3);
    border-radius: 50%;
    border-top-color: #fff;
    animation: spin 1s ease-in-out infinite;
    margin-bottom: 10px;
  }
  
  @keyframes spin {
    to { transform: rotate(360deg); }
  }
  
  /* ... rest of the styles remain unchanged ... */
  </style>