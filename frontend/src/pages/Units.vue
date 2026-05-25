<template>
  <LayoutHeader>
    <template #left-header>
      <ViewBreadcrumbs v-model="viewControls" routeName="Units" />
    </template>
    <template #right-header>
      <CustomActions
        v-if="unitsListView?.customListActions"
        :actions="unitsListView.customListActions"
      />
      <Button
        variant="solid"
        :label="__('Create')"
        iconLeft="plus"
        @click="showUnitModal = true"
      />
    </template>
  </LayoutHeader>
  <ViewControls
    ref="viewControls"
    v-model="units"
    v-model:loadMore="loadMore"
    v-model:resizeColumn="triggerResize"
    v-model:updatedPageCount="updatedPageCount"
    doctype="FCRM Unit"
  />
  <UnitsListView
    v-if="units.data && rows.length"
    ref="unitsListView"
    v-model="units.data.page_length_count"
    v-model:list="units"
    :rows="rows"
    :columns="columns"
    :options="{
      showTooltip: false,
      resizeColumn: true,
      rowCount: units.data.row_count,
      totalCount: units.data.total_count,
    }"
    @loadMore="() => loadMore++"
    @columnWidthUpdated="() => triggerResize++"
    @updatePageCount="(count) => (updatedPageCount = count)"
    @applyFilter="(data) => viewControls.applyFilter(data)"
    @applyLikeFilter="(data) => viewControls.applyLikeFilter(data)"
    @likeDoc="(data) => viewControls.likeDoc(data)"
    @selectionsChanged="
      (selections) => viewControls.updateSelections(selections)
    "
  />
  <EmptyState
    v-else-if="units.data && !rows.length"
    name="Units"
    :icon="UnitsIcon"
  />
  <UnitModal v-if="showUnitModal" v-model="showUnitModal" />
</template>

<script setup>
import ViewBreadcrumbs from '@/components/ViewBreadcrumbs.vue'
import CustomActions from '@/components/CustomActions.vue'
import UnitsIcon from '@/components/Icons/UnitsIcon.vue'
import LayoutHeader from '@/components/LayoutHeader.vue'
import UnitModal from '@/components/Modals/UnitModal.vue'
import UnitsListView from '@/components/ListViews/UnitsListView.vue'
import ViewControls from '@/components/ViewControls.vue'
import EmptyState from '@/components/ListViews/EmptyState.vue'
import { getMeta } from '@/stores/meta'
import { formatDate, timeAgo } from '@/utils'
import { ref, computed } from 'vue'

const { getFormattedPercent, getFormattedFloat, getFormattedCurrency } =
  getMeta('FCRM Unit')

const unitsListView = ref(null)
const showUnitModal = ref(false)

// units data is loaded in the ViewControls component
const units = ref({})
const loadMore = ref(1)
const triggerResize = ref(1)
const updatedPageCount = ref(20)
const viewControls = ref(null)

const rows = computed(() => {
  if (
    !units.value?.data?.data ||
    !['list', 'group_by'].includes(units.value.data.view_type)
  )
    return []
  return units.value.data.data.map((unit) => {
    let _rows = {}
    units.value.data.rows.forEach((row) => {
      _rows[row] = unit[row]

      let fieldType = units.value.data.columns?.find(
        (col) => (col.key || col.value) == row,
      )?.type

      if (
        fieldType &&
        ['Date', 'Datetime'].includes(fieldType) &&
        !['modified', 'creation'].includes(row)
      ) {
        _rows[row] = formatDate(unit[row], '', true, fieldType == 'Datetime')
      }

      if (fieldType && fieldType == 'Currency') {
        _rows[row] = getFormattedCurrency(row, unit)
      }

      if (fieldType && fieldType == 'Float') {
        _rows[row] = getFormattedFloat(row, unit)
      }

      if (fieldType && fieldType == 'Percent') {
        _rows[row] = getFormattedPercent(row, unit)
      }

      if (['modified', 'creation'].includes(row)) {
        _rows[row] = {
          label: formatDate(unit[row]),
          timeAgo: __(timeAgo(unit[row])),
        }
      }
    })
    return _rows
  })
})

const columns = computed(() => {
  let _columns = units.value?.data?.columns || []

  if (_columns.length) {
    _columns = _columns.map((col, index) => {
      if (index === _columns.length - 1) {
        return { ...col, align: 'right' }
      }
      return col
    })
  }

  return _columns
})
</script>
