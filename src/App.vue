<template>
  <div class="app">
    <h1>图片颜色拾取器</h1>
    <ImageColorPicker />
  </div>
</template>

<script setup>
import ImageColorPicker from './components/ImageColorPicker.vue'
</script>

<style>
.app {
  padding: 20px;
  max-width: 1200px;
  margin: 0 auto;
}

h1 {
  text-align: center;
  color: #333;
  margin-bottom: 30px;
}
</style> 