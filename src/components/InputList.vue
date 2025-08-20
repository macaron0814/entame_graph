<template>
  <div class="input-list-container">
    <div v-for="(item, index) in items" :key="item.id" class="input-item">
      <div class="percentage-display" :style="{ backgroundColor: item.color }">
        {{ Math.round(item.value) }}%
      </div>
      <input type="text" v-model="item.name" @input="emitUpdate" class="name-input" />
      <input
        type="range"
        min="0"
        max="100"
        step="1"
        :value="Math.round(item.value)"
        @input="handlePercentageChange(index, $event.target.value)"
        class="percentage-slider"
      />
      <button @click="removeItem(index)" class="remove-btn">×</button>
    </div>
    <button @click="addItem" class="add-btn">項目を追加</button>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue';

const emit = defineEmits(['update-data']);

const initialItems = [
  { id: 1, name: '根性', value: 20, color: '#41B883' },
  { id: 2, name: '学力', value: 20, color: '#E46651' },
  { id: 3, name: 'トーク力', value: 20, color: '#00D8FF' },
  { id: 4, name: 'バラエティ', value: 20, color: '#DD1B16' },
  { id: 5, name: '運', value: 20, color: '#F7B74E' },
];

const items = ref(JSON.parse(JSON.stringify(initialItems)));

function emitUpdate() {
  emit('update-data', items.value);
}

function handlePercentageChange(changedIndex, newValueStr) {
  const newValue = Number(newValueStr);
  items.value[changedIndex].value = newValue;

  let otherItemsSum = 0;
  items.value.forEach((item, index) => {
    if (index !== changedIndex) {
      otherItemsSum += item.value;
    }
  });

  const newOtherItemsSum = 100 - newValue;

  if (otherItemsSum === 0) {
    if (items.value.length > 1) {
      const equalValue = newOtherItemsSum / (items.value.length - 1);
      items.value.forEach((item, index) => {
        if (index !== changedIndex) {
          item.value = equalValue;
        }
      });
    }
  } else {
    const ratio = newOtherItemsSum / otherItemsSum;
    items.value.forEach((item, index) => {
      if (index !== changedIndex) {
        item.value *= ratio;
      }
    });
  }

  normalizePercentages();
  emitUpdate();
}

function addItem() {
  const newId = items.value.length > 0 ? Math.max(...items.value.map(i => i.id)) + 1 : 1;
  const newColor = `#${Math.floor(Math.random()*16777215).toString(16).padStart(6, '0')}`;
  items.value.push({ id: newId, name: '新しい項目', value: 0, color: newColor });
  redistributePercentages();
  emitUpdate();
}

function removeItem(index) {
  items.value.splice(index, 1);
  redistributePercentages();
  emitUpdate();
}

function redistributePercentages() {
    if (items.value.length === 0) return;
    const total = 100;
    const equalValue = total / items.value.length;
    items.value.forEach(item => {
        item.value = equalValue;
    });
    normalizePercentages();
}

function normalizePercentages() {
    if (items.value.length === 0) return;
    let sum = items.value.reduce((s, i) => s + i.value, 0);
    if (sum === 0) {
        redistributePercentages();
        return;
    }
    const factor = 100 / sum;
    items.value.forEach(item => {
        item.value *= factor;
    });

    let roundedSum = 0;
    for(let i = 0; i < items.value.length - 1; i++) {
        const rounded = Math.round(items.value[i].value);
        roundedSum += rounded;
        items.value[i].value = rounded;
    }
    items.value[items.value.length-1].value = 100 - roundedSum;
}

onMounted(() => {
  emitUpdate();
});

</script>

<style scoped>
.input-list-container {
  display: flex;
  flex-direction: column;
  gap: 1rem;
}

.input-item {
  display: flex;
  align-items: center;
  background: #2a2a2a;
  padding: 0.5rem 1rem;
  border-radius: 8px;
  gap: 1rem;
}

.percentage-display {
  font-size: 1.5rem;
  font-weight: bold;
  color: #fff;
  padding: 0.5rem 1rem;
  border-radius: 4px;
  min-width: 80px;
  text-align: center;
  font-family: 'Oswald', sans-serif;
}

.name-input {
  flex-grow: 1;
  background: #fff;
  border: none;
  padding: 0.8rem;
  border-radius: 4px;
  color: #111;
}

.percentage-slider {
  width: 100px;
  cursor: pointer;
}

.remove-btn {
  background: #ff4d4d;
  color: white;
  border: none;
  border-radius: 50%;
  width: 24px;
  height: 24px;
  font-size: 16px;
  cursor: pointer;
  line-height: 24px;
  text-align: center;
}

.add-btn {
  background: #2196f3;
  color: #fff;
  border: none;
  padding: 0.8rem;
  border-radius: 8px;
  cursor: pointer;
  font-size: 1rem;
  margin-top: 1rem;
}
</style>
