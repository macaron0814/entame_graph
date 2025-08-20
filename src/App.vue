<template>
  <div id="app-layout">
    <header class="header-bar">
      <div class="title">NOGIZAKA UNDER CONSTRUCTION</div>
      <div class="profile">
        <span>FILE NO.02</span>
        <span>賀喜遥香</span>
        <span>AGE 24</span>
      </div>
    </header>
    <main class="main-content">
      <div class="chart-area">
        <DoughnutChart :chart-data="chartData" :image-url="imageUrl" />
        <div class="uploader-container" v-if="!imageUrl">
          <ImageUploader @image-uploaded="onImageUploaded" />
        </div>
      </div>
      <div class="input-sidebar">
        <InputList @update-data="onDataUpdated" />
      </div>
    </main>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue';
import DoughnutChart from './components/DoughnutChart.vue';
import InputList from './components/InputList.vue';
import ImageUploader from './components/ImageUploader.vue';

const items = ref([]);
const imageUrl = ref(null);

const chartData = computed(() => {
  if (!items.value || items.value.length === 0) {
    return null;
  }
  return {
    labels: items.value.map(item => item.name),
    datasets: [
      {
        backgroundColor: items.value.map(item => item.color),
        data: items.value.map(item => item.value),
      },
    ],
  };
});

function onDataUpdated(newItems) {
  items.value = newItems;
}

function onImageUploaded(url) {
  imageUrl.value = url;
}
</script>

<style>
@import url('https://fonts.googleapis.com/css2?family=Oswald:wght@700&display=swap');

body {
  margin: 0;
  background-color: #111;
  color: #fff;
  font-family: sans-serif;
}

#app-layout {
  display: flex;
  flex-direction: column;
  height: 100vh;
}

.header-bar {
  background: #000;
  color: #fff;
  padding: 1rem 2rem;
  display: flex;
  justify-content: space-between;
  align-items: center;
  font-family: 'Oswald', sans-serif;
  border-bottom: 2px solid #333;
}

.header-bar .title {
  font-size: 2rem;
}

.header-bar .profile span {
  margin-left: 1.5rem;
  font-size: 1.2rem;
}

.main-content {
  display: flex;
  flex-grow: 1;
  padding: 2rem;
  gap: 2rem;
}

.chart-area {
  flex: 1;
  display: flex;
  justify-content: center;
  align-items: center;
  position: relative;
}

.input-sidebar {
  width: 400px;
  background: #000;
  padding: 2rem;
  border-left: 2px solid #333;
  overflow-y: auto;
}

.uploader-container {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  z-index: 10;
}
</style>
