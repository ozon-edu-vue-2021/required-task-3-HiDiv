<template>
  <div class="menu">
    <div class="toolbar">
      <div class="toolbar__header">
        <template v-if="!isUserOpened">
          <h3>Информация</h3>
        </template>
        <template v-else>
          <div class="action">
            <div class="arrow" @click="closeProfile"></div>
          </div>
          <h3>Профиль</h3>
        </template>
      </div>
      <div class="toolbar__actions"></div>
    </div>
    <div class="content">
      <div v-if="!isUserOpened" class="legend">
        <div class="legend__data">
          <div v-if="localLegend.length > 0" class="legend__items">
            <Draggable v-model="localLegend">
              <LegendItem
                v-for="(item, index) in localLegend"
                :key="index"
                :color="item.color"
                :text="item.text"
                :counter="item.counter"
                class="legend__item"
                @changeColor="changeGroupColor({ oldColor: $event, idx: index })"
              />
            </Draggable>
          </div>
          <span v-else class="legend--empty"> Список пуст </span>
        </div>
        <div class="legend__chart">
          <div>
            {{ formattedDate }}
          </div>

          <PieChart :chart-data="legendChartData" :options="{ borderWidth: '10px', legend: { display: false } }" />
        </div>
      </div>
      <div v-else class="profile">
        <div v-if="!person" class="profile__empty">Место пустое</div>

        <PersonCard :person="person" @close="closeProfile" />
      </div>
    </div>

    <group-color-picker
      id="group-color-picker"
      :group-id="editedGroupId"
      :group-name="editedGroupName"
      :color="editedColor"
      @change="groupColorChanged"
    ></group-color-picker>
  </div>
</template>

<script>
import LegendItem from './SideMenu/LegendItem.vue'
import PersonCard from './SideMenu/PersonCard.vue'
import GroupColorPicker from './SideMenu/GroupColorPicker.vue'
import PieChart from './SideMenu/PieChart.vue'

import Draggable from 'vuedraggable'
import { format } from 'date-fns'

export default {
  props: {
    legend: {
      type: Array,
      required: true,
    },
    isUserOpened: {
      type: Boolean,
      default: false,
    },
    person: {
      type: Object,
      default: null,
    },
  },
  components: {
    LegendItem,
    PersonCard,
    Draggable,
    PieChart,
    GroupColorPicker,
  },
  data: () => ({
    editedGroupName: '',
    editedColor: '#000000',
    editedGroupId: 0,
  }),
  methods: {
    changeGroupColor(event) {
      this.editedColor = event.oldColor
      this.editedGroupName = this.legend[event.idx].text
      this.editedGroupId = this.legend[event.idx].group_id
      this.$bvModal.show('group-color-picker')
    },
    groupColorChanged(event) {
      const idx = this.legend.findIndex((group) => group.group_id === event.groupId)
      if (~idx) {
        this.localLegend[idx].color = event.color
      }
    },

    closeProfile() {
      this.$emit('update:isUserOpened', false)
    },
  },
  computed: {
    localLegend: {
      get() {
        return this.legend
      },
      set(newValue) {
        this.$emit('update:legend', newValue)
      },
    },
    formattedDate() {
      return format(new Date(), 'dd.MM.yyyy hh:mm')
    },
    legendChartData() {
      return {
        labels: this.legend.map((it) => it.text),
        datasets: [
          {
            label: 'Легенда',
            backgroundColor: this.legend.map((legendItem) => legendItem.color),
            data: this.legend.map((legendItem) => legendItem.counter),
          },
        ],
      }
    },
  },
}
</script>

<style scoped>
.menu {
  border-left: 1px solid #ccd8e4;
  padding: 24px;
  display: flex;
  flex-direction: column;
  overflow: hidden;
}

.toolbar {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.toolbar .toolbar__actions button {
  font-size: 0.76rem;
  text-transform: uppercase;
  letter-spacing: 0.08rem;
  padding: 2px 6px;
}

.toolbar__header {
  display: flex;
  align-items: center;
  margin-bottom: 12px;
}

.toolbar__header .action {
  cursor: pointer;
  margin-right: 14px;
  width: 20px;
  height: 20px;
  display: flex;
  justify-content: center;
  align-items: center;
}

.toolbar__header .action .arrow {
  width: 10px;
  height: 10px;
  border-top: 2px solid blue;
  border-right: 2px solid blue;
  transform: rotate(-135deg);
}

h3 {
  margin: 0;
}

.content {
  flex: 1;
}

.content .legend {
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  height: 100%;
}

.content .legend .legend__data {
  display: flex;
  height: 100%;
}

.content .legend .legend__items {
  flex: 1;
  width: 100%;
}

.content .legend .legend__items .legend__item:not(:first-child) {
  margin-top: 16px;
}

.content .legend .legend__items .legend__item {
  cursor: pointer;
}

.content .legend .legend__items .legend__item.sortable-chosen {
  opacity: 25%;
}

.content .legend .legend--empty {
  align-self: center;
  width: 100%;
  text-align: center;
}

.profile {
  padding-top: 20px;
}
</style>
