<script setup lang="ts">
import { ref, h } from 'vue'
import { NSwitch, NDataTable } from 'naive-ui'

interface TableRow {
  id: number
  steName: string
  isActual: boolean
  priceEndDate: number
  priceNotNds: number
  nds: number
  price: number
  fillEndDate: string
}

const TABLE_HEADERS = [
  {
    key: "steName",
    title: "Наименование СТЕ",
    sorter: true,
  },
  {
    key: 'isActual',
    title: 'В наличии',
    render(row: TableRow) {
      return h(NSwitch, {
        value: row.isActual
      })
    }
  },
  {
    key: "priceEndDate",
    title: "Срок действия предоставленных сведений",
  },
  {
    key: "priceNotNds",
    title: "Цена, руб. без НДС",
  },
  {
    key: "nds",
    title: "НДС, %",
  },
  {
    key: "price",
    title: "Цена, руб. с НДС",

  },
  {
    key: "fillEndDate",
    title: "Срок заполнения",
  },
];

const tableData = ref([
  {
    id: 1,
    steName: 'СТЕ №1',
    isActual: true,
    priceEndDate: Date.now(),
    priceNotNds: 1000,
    nds: 20,
    price: 1200,
    fillEndDate: '2025-12-01',
  },
  {
    id: 2,
    steName: 'СТЕ №2',
    isActual: false,
    priceEndDate: Date.now(),
    priceNotNds: 2000,
    nds: 10,
    price: 2200,
    fillEndDate: '2025-11-15',
  },
])
</script>

<template>
  <div class="custom-table-wrapper">
    <n-data-table size="large"
        :columns="TABLE_HEADERS"
        :data="tableData"
        class="custom-table"
    />
  </div>
</template>

<style scoped>
.custom-table-wrapper {
  padding: 24px;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.06);
}

.custom-table :deep(.n-data-table) {
  width: 100%;
  height: 100%;
  font-family: 'Segoe UI', sans-serif;
  font-size: 14px;
}

.custom-table :deep(.n-data-table-th) {
  background-color: #5483e6;
  font-weight: 600;
  color: #ffffff;
  text-transform: uppercase;
}

.custom-table :deep(.n-data-table-td) {
  padding: 12px;
}

.custom-table :deep(.n-data-table-tr:hover) {
  background-color: #1662f3;
}

.custom-table :deep(.n-data-table-base-table) {
  overflow: hidden;
}
</style>