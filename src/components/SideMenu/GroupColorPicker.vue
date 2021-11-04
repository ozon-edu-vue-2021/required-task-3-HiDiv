<template>
  <b-modal :id="$attrs.id" centered :title="'Выберете новый цвет для группы ' + groupName" @ok="updateColor">
    <chrome-picker v-model="newColor"></chrome-picker>
  </b-modal>
</template>

<script>
import { Chrome } from 'vue-color'

export default {
  name: 'GroupColorPicker',
  inheritAttrs: false,
  components: {
    'chrome-picker': Chrome,
  },
  props: {
    groupId: {
      type: Number,
      required: true,
    },
    groupName: {
      type: String,
      default: '',
    },
    color: {
      type: String,
      default: '#000000',
    },
  },
  data() {
    return {
      newColor: this.color,
    }
  },
  methods: {
    updateColor() {
      this.$emit('change', { groupId: this.groupId, color: this.newColor.hex })
    },
  },
  watch: {
    color() {
      this.newColor = this.color
    },
  },
}
</script>

<style scoped></style>
