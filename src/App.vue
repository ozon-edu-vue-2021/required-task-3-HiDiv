<template>
  <div id="app">
    <div class="office">
      <Map
        :tables.sync="tables"
        :legend.sync="legend"
        :people.sync="people"
        v-on:table-click="onTableClick"
      />
      <SideMenu :legend.sync="legend" :person="person" :is-user-opened.sync="isUserOpened" />
    </div>
  </div>
</template>

<script>
import Map from './components/Map.vue'
import SideMenu from './components/SideMenu.vue'

import tables from '@/assets/data/tables.json'
import legend from '@/assets/data/legend.json'
import people from '@/assets/data/people.json'

export default {
  name: 'App',
  components: {
    Map,
    SideMenu,
  },
  data: () => ({
    tables: tables,
    legend: legend,
    people: people,
    isUserOpened: false,
    person: {},
  }),
  methods: {
    onTableClick(tableId) {
      const idx = this.people.findIndex((item) => item.tableId === tableId)
      if (~idx) {
        this.person = this.people[idx]
        this.isUserOpened = true
      }
    },
  },
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  color: #2c3e50;
  background-color: #fafafa;
  padding: 24px;
  box-sizing: border-box;
}

html,
body,
#app {
  height: 100%;
}

* {
  box-sizing: border-box;
}

h3 {
  margin-top: 0px;
}

.office {
  display: grid;
  grid-template-columns: 1fr 320px;
  border-radius: 6px;
  border: 1px solid #ccd8e4;
  height: 100%;
  background: white;
  max-width: 1500px;
  margin: 0 auto;
}
</style>
