<template>
  <div
    class="upload-area"
    @dragover.prevent="onDragOver"
    @dragleave.prevent="onDragLeave"
    @drop.prevent="onDrop"
    @click="onClick"
  >
    <input type="file" ref="fileInput" @change="onFileChange" accept="image/*" style="display: none;" />
    <div v-if="!isDragging">
      <p>画像をドラッグ＆ドロップ</p>
      <p>またはクリックして選択</p>
    </div>
    <div v-else class="dragging-overlay">
      <p>ここにドロップ</p>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue';

const emit = defineEmits(['image-uploaded']);
const fileInput = ref(null);
const isDragging = ref(false);

function handleFile(file) {
  if (!file || !file.type.startsWith('image/')) {
    return;
  }
  const reader = new FileReader();
  reader.onload = (e) => {
    emit('image-uploaded', e.target.result);
  };
  reader.readAsDataURL(file);
}

function onDragOver(e) {
  isDragging.value = true;
}

function onDragLeave(e) {
  isDragging.value = false;
}

function onDrop(e) {
  isDragging.value = false;
  const file = e.dataTransfer.files[0];
  handleFile(file);
}

function onClick() {
  fileInput.value.click();
}

function onFileChange(e) {
  const file = e.target.files[0];
  handleFile(file);
}
</script>

<style scoped>
.upload-area {
  width: 200px;
  height: 200px;
  border: 2px dashed #ccc;
  border-radius: 50%;
  display: flex;
  justify-content: center;
  align-items: center;
  text-align: center;
  color: #ccc;
  cursor: pointer;
  transition: background-color 0.2s, border-color 0.2s;
  position: relative; /* For the overlay */
}

.upload-area:hover {
  border-color: #fff;
  background-color: #333;
}

.dragging-overlay {
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    background-color: rgba(0, 123, 255, 0.5);
    display: flex;
    justify-content: center;
    align-items: center;
    border-radius: 50%;
}
</style>
