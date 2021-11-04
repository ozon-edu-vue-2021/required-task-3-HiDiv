<template>
  <div class="legend-item">
    <div class="legend-item__marker">
      <div class="marker" :style="{ '--marker-color': color }" @click="onMarkerClick" />
    </div>
    <div class="legend-item__label">{{ text }}</div>
    <div class="legend-item__counter">{{ counter }}</div>
  </div>
</template>

<script>
const rgbToHex = (r, g, b) => '#' + ((1 << 24) + (r << 16) + (g << 8) + b).toString(16).slice(1)

export default {
  name: 'LegendItem',
  props: {
    color: {
      type: String,
      default: '#fff',
    },
    text: {
      type: String,
      required: true,
    },
    counter: {
      type: Number,
      default: 0,
    },
  },
  methods: {
    // Преобразуем фактический цвет маркера в hex-форму
    onMarkerClick(nativeEvent) {
      const colorAsRgbStr = getComputedStyle(nativeEvent.target).backgroundColor
      const colorParts = colorAsRgbStr.match(/(\d{1,3}),\s*(\d{1,3}),\s*(\d{1,3})/)
      const hexColor = colorParts ? rgbToHex(+colorParts[1], +colorParts[2], +colorParts[3]) : '#000000'
      this.$emit('changeColor', hexColor)
    },
  },
}
</script>

<style scoped>
.legend-item {
  display: flex;
  align-items: center;
}

.legend-item .legend-item__marker .marker {
  width: 18px;
  height: 18px;
  background-color: var(--marker-color);
  border: 1px solid #bababa;
  border-radius: 5px;
}

.legend-item .legend-item__label {
  flex-basis: 100%;
  margin: 0 24px;
}

.legend-item .legend-item__counter {
  color: #8e8e8e;
  font-size: 0.9rem;
}
</style>
