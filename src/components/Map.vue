<template>
  <div class="map">
    <h3>Карта офиса</h3>

    <div v-if="!isLoading" class="map-root">
      <MapSVG ref="svg" @click="beginCreatePeople" />
      <Table v-show="false" ref="table" />
    </div>
    <div v-else>Loading...</div>

    <edit-people
      id="create-people"
      title="Добавить нового сотрудника"
      :legend="legend"
      @change="endCreatePeople"
    ></edit-people>
  </div>
</template>

<script>
import * as d3 from 'd3'

import MapSVG from '@/assets/images/map.svg'
import Table from '@/assets/images/workPlace.svg'
import EditPeople from '@/components/EditPeople.vue'

// Преобразует координаты клика на экране в относительные координаты svg
const getMousePosInSvg = (event) => {
  const ctm = event.target.getScreenCTM()

  return {
    x: (event.clientX - ctm.e) / ctm.a,
    y: (event.clientY - ctm.f) / ctm.d,
  }
}

export default {
  components: {
    MapSVG,
    Table,
    EditPeople,
  },
  props: {
    tables: {
      type: Array,
      required: true,
    },
    legend: {
      type: Array,
      required: true,
    },
    people: {
      type: Array,
      required: true,
    },
  },
  data() {
    return {
      isLoading: false,
      svg: null,
      g: null,
      tableSVG: null,
      newTableXY: null,
    }
  },
  mounted() {
    this.isLoading = true

    this.svg = d3.select(this.$refs.svg)
    this.g = this.svg.select('g')
    this.tableSVG = d3.select(this.$refs.table)

    if (!this.g) {
      throw new Error('SVG is incorrect')
    }

    this.drawTables()

    this.isLoading = false
  },
  methods: {
    drag() {
      const vm = this
      let tableId, offsetX, offsetY

      const realX = (event) => event.x - offsetX
      const realY = (event) => event.y - offsetY

      function dragStarted(event) {
        const table = d3.select(this)
        tableId = table.attr('id')
        offsetX = event.x - vm.tableById[tableId].x
        offsetY = event.y - vm.tableById[tableId].y

        table.raise().attr('stroke', 'gray')
      }

      function dragged(event) {
        const x = realX(event)
        const y = realY(event)
        d3.select(this).attr('transform', `translate(${x}, ${y}) scale(0.5)`)
      }

      function dragEnded(event) {
        d3.select(this).attr('stroke', null)

        vm.updateTable(tableId, realX(event), realY(event))
      }

      return d3.drag().on('start', dragStarted).on('drag', dragged).on('end', dragEnded)
    },
    updateTable(tableId, newX, newY) {
      const idx = this.tableById[tableId]?.idx ?? -1
      if (~idx) {
        const newTables = this.tables.slice()
        newTables[idx].x = newX
        newTables[idx].y = newY
        this.$emit('update:tables', newTables)
      }
    },
    drawTables() {
      const vm = this
      const svgTablesGroupPlace = this.g.append('g').classed('groupPlaces', true)

      this.tables.map((table) => {
        const targetSeat = svgTablesGroupPlace
          .append('g')
          .attr('transform', `translate(${table.x}, ${table.y}) scale(0.5)`)
          .attr('id', table._id)
          .classed('employer-place', true)

        targetSeat
          .append('g')
          .classed('employer-table', true)
          .attr('transform', `rotate(${table.rotate || 0})`)
          .attr('group_id', table.group_id)
          .classed('table', true)
          .html(this.tableSVG.html())

        targetSeat.call(this.drag()).on('click', function (event) {
          event.stopPropagation()
          vm.onTableClick(+d3.select(this).attr('id'))
        })
      })

      this.fillTablesColor()
    },
    // Вынесем покраску столов отдельным методом, чтобы не перерисовывать всю карту
    fillTablesColor() {
      d3.selectAll('.employer-table').attr('fill', 'transparent')
      Object.keys(this.colorForGroup).forEach((groupId) =>
        d3.selectAll('.employer-table[group_id="' + groupId + '"]').attr('fill', this.colorForGroup[groupId])
      )
    },

    onTableClick(tableId) {
      this.$emit('table-click', tableId)
    },

    beginCreatePeople(event) {
      const point = getMousePosInSvg(event)
      const viewBox = this.$refs.svg.viewBox.baseVal
      if (
        point.x < viewBox.x ||
        point.y < viewBox.y ||
        point.x > viewBox.x + viewBox.width ||
        point.y > viewBox.y + viewBox.height
      ) {
        return
      }
      this.newTableXY = Object.assign({}, point)

      this.$bvModal.show('create-people')
    },
    endCreatePeople(createdPeople) {
      // Добавляем новый стол
      const newTable = {
        _id: this.getNewTableId(),
        x: this.newTableXY.x,
        y: this.newTableXY.y,
        group_id: createdPeople.groupId,
        rotate: 80,
      }
      this.localTables.push(newTable)

      // Обновляем количество столов в подразделении
      // eslint-disable-next-line prettier/prettier
      const legendIdx = this.localLegend.findIndex((item) => item.group_id === createdPeople.groupId)
      if (~legendIdx) {
        this.localLegend[legendIdx].counter++
      }

      // Обновляем сотрудников
      delete createdPeople.groupId
      createdPeople._id = this.getNewPeopleId()
      createdPeople.tableId = newTable._id
      this.localPeople.push(createdPeople)
    },
    getNewTableId() {
      // eslint-disable-next-line prettier/prettier
      const maxTableId = this.tables.reduce((maxId, table) => table._id > maxId ? table._id : maxId, 0)
      return maxTableId + 1
    },
    getNewPeopleId() {
      // eslint-disable-next-line prettier/prettier
      const maxId = this.people.reduce((maxId, item) => item._id > maxId ? item._id : maxId, 0)
      return maxId + 1
    },
  },
  computed: {
    // Вспомогательная структура для маппинга цвета на id группы.
    // Ускоряет поиск цвета для группы и позволяет отслеживать изменения цвета группы в watch без применения deep.
    colorForGroup() {
      return this.legend.reduce((x, group) => ({ ...x, [group.group_id]: group?.color ?? 'transparent' }), {})
    },
    // Вспомогательная структура для маппинга столов
    tableById() {
      return this.tables.reduce(
        (x, table, idx) => ({ ...x, [table._id]: { x: table.x, y: table.y, groupId: table.group_id, idx } }),
        {}
      )
    },

    localTables: {
      get() {
        return this.tables
      },
      set(newTables) {
        this.$emit('update:tables', newTables)
      },
    },
    localLegend: {
      get() {
        return this.legend
      },
      set(newLegend) {
        this.$emit('update:legend', newLegend)
      },
    },
    localPeople: {
      get() {
        return this.people
      },
      set(newPeople) {
        this.$emit('update:people', newPeople)
      },
    },
  },
  watch: {
    colorForGroup: 'fillTablesColor',
    tableById: 'drawTables',
  },
}
</script>

<style scoped>
.map {
  height: 100%;
  width: 100%;
  padding: 24px;
  overflow: hidden;
  box-sizing: border-box;
  display: flex;
  flex-direction: column;
}

.map-root {
  height: 100%;
  width: 100%;
  overflow: hidden;
  box-sizing: border-box;
}

h3 {
  margin-top: 0;
}

::v-deep svg {
  height: 100%;
  width: 100%;
}

::v-deep .table {
  cursor: pointer;
}
</style>
