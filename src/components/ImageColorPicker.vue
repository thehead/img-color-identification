<template>
  <div class="color-picker-container">
    <div class="upload-section">
      <input
        type="file"
        accept="image/*"
        @change="handleImageUpload"
        class="file-input"
      />
      <div class="preview-area" ref="previewArea">
        <img
          v-if="imageUrl"
          :src="imageUrl"
          @mousedown="startDragging"
          @mousemove="handleDrag"
          @mouseup="stopDragging"
          @mouseleave="stopDragging"
          ref="previewImage"
          class="preview-image"
        />
        <div 
          v-if="isDragging" 
          class="magnifier"
          :style="{
            left: `${cursorPos.x + 20}px`,
            top: `${cursorPos.y}px`
          }"
        >
          <canvas ref="magnifierCanvas" width="100" height="100"></canvas>
          <div class="color-text">
            RGB: {{ selectedColor?.r }}, {{ selectedColor?.g }}, {{ selectedColor?.b }}
          </div>
        </div>
      </div>
    </div>
    
    <div class="color-info" v-if="selectedColor">
      <div 
        class="color-preview" 
        :style="{ backgroundColor: `rgb(${selectedColor.r}, ${selectedColor.g}, ${selectedColor.b})` }"
      ></div>
      <div class="color-values">
        <p>RGB: {{ selectedColor.r }}, {{ selectedColor.g }}, {{ selectedColor.b }}</p>
        <p>HEX: {{ selectedColor.hex }}</p>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue'

const imageUrl = ref('')
const selectedColor = ref(null)
const previewImage = ref(null)
const previewArea = ref(null)
const magnifierCanvas = ref(null)

const isDragging = ref(false)
const cursorPos = ref({ x: 0, y: 0 })

const handleImageUpload = (event) => {
  const input = event.target
  if (input.files && input.files[0]) {
    const file = input.files[0]
    imageUrl.value = URL.createObjectURL(file)
  }
}

const rgbToHex = (r, g, b) => {
  return '#' + [r, g, b]
    .map(x => {
      const hex = x.toString(16)
      return hex.length === 1 ? '0' + hex : hex
    })
    .join('')
}

const startDragging = (event) => {
  isDragging.value = true
  handleDrag(event)
}

const stopDragging = () => {
  isDragging.value = false
}

const handleDrag = (event) => {
  if (!isDragging.value) return

  const rect = previewImage.value.getBoundingClientRect()
  cursorPos.value = {
    x: event.clientX - rect.left,
    y: event.clientY - rect.top
  }

  const canvas = document.createElement('canvas')
  const context = canvas.getContext('2d')
  if (!context) return

  canvas.width = previewImage.value.naturalWidth
  canvas.height = previewImage.value.naturalHeight
  context.drawImage(previewImage.value, 0, 0)

  const scaleX = previewImage.value.naturalWidth / rect.width
  const scaleY = previewImage.value.naturalHeight / rect.height
  const x = Math.round(cursorPos.value.x * scaleX)
  const y = Math.round(cursorPos.value.y * scaleY)

  const pixel = context.getImageData(x, y, 1, 1).data
  selectedColor.value = {
    r: pixel[0],
    g: pixel[1],
    b: pixel[2],
    hex: rgbToHex(pixel[0], pixel[1], pixel[2])
  }

  updateMagnifier(context, x, y)
}

const updateMagnifier = (sourceContext, x, y) => {
  const magCtx = magnifierCanvas.value.getContext('2d')
  if (!magCtx) return

  // 清除画布
  magCtx.clearRect(0, 0, 100, 100)
  
  const size = 10 // 采样区域大小
  const sampleX = Math.max(0, Math.min(x - size/2, previewImage.value.naturalWidth - size))
  const sampleY = Math.max(0, Math.min(y - size/2, previewImage.value.naturalHeight - size))
  
  // 获取采样区域的图像数据
  const imageData = sourceContext.getImageData(
    sampleX,
    sampleY,
    size,
    size
  )

  // 创建临时画布来处理放大效果
  const tempCanvas = document.createElement('canvas')
  tempCanvas.width = size
  tempCanvas.height = size
  const tempCtx = tempCanvas.getContext('2d')
  tempCtx.putImageData(imageData, 0, 0)

  // 清除主画布并绘制放大的图像
  magCtx.imageSmoothingEnabled = false
  magCtx.drawImage(
    tempCanvas,
    0, 0, size, size,  // 源图像区域
    0, 0, 100, 100     // 目标区域（放大到100x100）
  )
  
  // 绘制十字准星
  magCtx.strokeStyle = 'rgba(255,255,255,0.8)'
  magCtx.lineWidth = 1
  magCtx.beginPath()
  magCtx.moveTo(50, 0)
  magCtx.lineTo(50, 100)
  magCtx.moveTo(0, 50)
  magCtx.lineTo(100, 50)
  magCtx.stroke()

  // 在中心点绘制一个小方框标记当前像素
  magCtx.strokeStyle = 'rgba(255,0,0,0.8)'
  magCtx.strokeRect(45, 45, 10, 10)
}
</script>

<style scoped>
.color-picker-container {
  max-width: 800px;
  margin: 0 auto;
  padding: 20px;
}

.upload-section {
  margin-bottom: 20px;
}

.preview-area {
  margin-top: 20px;
  border: 2px dashed #ccc;
  min-height: 200px;
  display: flex;
  align-items: center;
  justify-content: center;
  position: relative;
}

.preview-image {
  max-width: 100%;
  max-height: 500px;
  cursor: crosshair;
  user-select: none;
  -webkit-user-drag: none;
}

.color-info {
  display: flex;
  align-items: center;
  gap: 20px;
  padding: 20px;
  background: #f5f5f5;
  border-radius: 8px;
}

.color-preview {
  width: 50px;
  height: 50px;
  border-radius: 8px;
  border: 2px solid #ddd;
}

.color-values {
  font-family: monospace;
}

.magnifier {
  position: absolute;
  width: 100px;
  height: 120px;
  background: white;
  border: 2px solid #333;
  border-radius: 8px;
  pointer-events: none;
  z-index: 1000;
}

.magnifier canvas {
  width: 100px;
  height: 100px;
  image-rendering: pixelated;
}

.color-text {
  font-size: 10px;
  text-align: center;
  padding: 2px;
  background: rgba(0,0,0,0.8);
  color: white;
}
</style> 