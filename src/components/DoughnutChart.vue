<template>
  <div class="chart-container">
    <Doughnut v-if="chartData" :data="chartData" :options="chartOptions" :plugins="plugins" />
  </div>
</template>

<script setup>
import { ref, watch, computed } from 'vue';
import { Doughnut } from 'vue-chartjs';
import { Chart as ChartJS, Title, Tooltip, Legend, ArcElement } from 'chart.js';

ChartJS.register(Title, Tooltip, Legend, ArcElement);

const props = defineProps({
  chartData: {
    type: Object,
    required: true,
  },
  imageUrl: {
    type: String,
    default: null,
  },
});

const loadedImage = ref(null);

watch(() => props.imageUrl, (newUrl) => {
  if (newUrl) {
    const image = new Image();
    image.src = newUrl;
    image.onload = () => {
      loadedImage.value = image;
    };
  } else {
    loadedImage.value = null;
  }
}, { immediate: true });

const centerImagePlugin = {
  id: 'centerImage',
  afterDraw: (chart) => {
    if (loadedImage.value) {
      const ctx = chart.ctx;
      const chartArea = chart.chartArea;
      if (!chartArea) return; // aint no chart area yet
      const centerX = (chartArea.left + chartArea.right) / 2;
      const centerY = (chartArea.top + chartArea.bottom) / 2;

      const doughnutRadius = chart.getDatasetMeta(0).data[0]?.innerRadius;
      if (!doughnutRadius) return;

      const imageSize = doughnutRadius * 2 * 0.9;

      ctx.save();
      ctx.beginPath();
      ctx.arc(centerX, centerY, imageSize / 2, 0, 2 * Math.PI);
      ctx.closePath();
      ctx.clip();

      ctx.drawImage(loadedImage.value, centerX - imageSize / 2, centerY - imageSize / 2, imageSize, imageSize);
      ctx.restore();
    }
  },
};

const plugins = computed(() => [centerImagePlugin]);

const chartOptions = ref({
  responsive: true,
  maintainAspectRatio: false,
  cutout: '70%',
  plugins: {
    legend: {
      display: false,
    },
    tooltip: {
        callbacks: {
            label: function(context) {
                let label = context.label || '';
                if (label) {
                    label += ': ';
                }
                if (context.parsed !== null) {
                    label += Math.round(context.parsed) + '%';
                }
                return label;
            }
        }
    }
  },
});
</script>

<style scoped>
.chart-container {
  position: relative;
  width: 600px;
  height: 600px;
}
</style>
