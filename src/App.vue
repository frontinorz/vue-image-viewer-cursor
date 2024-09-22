<template>
  <div>
    <h1>Vue Image Viewer</h1>
    <ImageViewer :images="imageUrls" />
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import ImageViewer from './components/ImageViewer.vue'

const imageUrls = ref([])

const fetchRandomImages = () => {
  const count = 10 // Number of images to fetch
  const imageSize = 800 // Size of the images (width and height)
  
  imageUrls.value = Array.from({ length: count }, (_, index) => 
    `https://picsum.photos/${imageSize}?random=${index}`
  )
}

onMounted(() => {
  fetchRandomImages()
})
</script>