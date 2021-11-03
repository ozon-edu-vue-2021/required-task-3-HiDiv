<template>
  <b-modal :id="$attrs.id" centered :title="title" @ok="updatePeople" @show="onReset">
    <b-form @reset="onReset">
      <b-form-group id="input-group-0" label="Подразделение:" label-for="input-0">
        <b-form-select
          class="form-control"
          id="input-0"
          v-model="localPeople.groupId"
          :options="groups"
        ></b-form-select>
      </b-form-group>

      <b-form-group id="input-group-1" label="ФИО:" label-for="input-1">
        <b-form-input id="input-1" v-model="localPeople.name" type="text" placeholder="Введите ФИО"></b-form-input>
      </b-form-group>

      <b-form-group id="input-group-2" label="Email адрес:" label-for="input-2">
        <b-form-input id="input-2" v-model="localPeople.email" type="email" placeholder="Введите email"></b-form-input>
      </b-form-group>

      <b-form-group id="input-group-3" label="Аватар:" label-for="input-3">
        <b-form-input
          id="input-3"
          v-model="localPeople.picture"
          type="url"
          placeholder="Введите url картинки с аватаром"
        ></b-form-input>
      </b-form-group>

      <b-form-group id="input-group-4" label="Описание:" label-for="input-4">
        <b-form-textarea
          id="input-4"
          v-model="localPeople.about"
          placeholder="Введите произвольное описание"
          rows="3"
          max-rows="6"
        ></b-form-textarea>
      </b-form-group>

      <b-form-group id="input-group-5" label="Возраст:" label-for="input-5">
        <b-form-input id="input-5" v-model="localPeople.age" type="number" placeholder="Введите возраст"></b-form-input>
      </b-form-group>

      <b-form-group id="input-group-6" label="Дата регистрации:" label-for="input-6">
        <b-form-input
          id="input-6"
          v-model="localPeople.registered"
          type="datetime"
          placeholder="Введите дату и время регистрации"
        ></b-form-input>
      </b-form-group>
    </b-form>
  </b-modal>
</template>

<script>
import { format } from 'date-fns'

export default {
  name: 'EditPeople',
  inheritAttrs: false,
  props: {
    title: {
      type: String,
      default: '',
    },
    people: {
      type: Object,
      default: () => ({}),
    },
    legend: {
      type: Array,
      required: true,
    },
  },
  data() {
    return {
      localPeople: {},
    }
  },
  created() {
    this.onReset()
  },
  methods: {
    onReset() {
      this.localPeople = {
        groupId: this.groupId ?? 0,
        name: this.name ?? '',
        email: this.email ?? '',
        picture: this.picture ?? '',
        about: this.about ?? '',
        age: this.age ?? null,
        registered: this.registered ?? format(new Date(), 'dd.MM.yyyy hh:mm'),
      }
    },
    updatePeople() {
      this.$emit('change', this.localPeople)
    },
  },
  computed: {
    groups() {
      return this.legend.map((item) => ({ value: item.group_id, text: item.text }))
    },
  },
}
</script>
