<template>
  <div class="person">
    <div class="person__photo">
      <img :src="person.picture" alt="photo" />
    </div>
    <div class="person__info">
      <div class="person__info-name">
        <b>{{ person.name }} ({{ person.age }})</b>
      </div>

      <div class="person__info-email">Почта: {{ person.email }}</div>
      <div class="person__info-email">Дата регистрации: {{ formattedDate }}</div>
      <div class="person__info-about">О себе: {{ person.about }}</div>
    </div>
  </div>
</template>

<script>
import { format } from 'date-fns'

export default {
  props: {
    person: {
      type: Object,
      default: null,
    },
  },
  mounted() {
    document.addEventListener('click', this.clickOutsideHandler)
  },
  beforeDestroy() {
    document.removeEventListener('click', this.clickOutsideHandler)
  },
  methods: {
    // Готовый плагин при данной архитектуре и верстке работает неправильно,
    // напишем собственную реализацию
    clickOutsideHandler(event) {
      if (!this.$el.contains(event.target)) {
        this.$emit('close')
      }
    },
  },
  computed: {
    formattedDate() {
      return format(new Date(this.person.registered), 'dd.MM.yyyy hh:mm')
    },
  },
}
</script>

<style scoped>
.person {
  display: grid;
  grid-template-columns: 50px 1fr;
  grid-gap: 8px;
}

.person__photo img {
  height: 50px;
  width: 50px;
  border-radius: 50%;
}

.person__info {
  display: grid;
  grid-gap: 8px;
}

.person__info-name {
  margin-bottom: 10px;
}
</style>
