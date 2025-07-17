<script setup lang="ts">
import { ref, h } from 'vue'
import {NSwitch, NDataTable, NDatePicker, NInput, NInputNumber} from 'naive-ui'
import DateInputCell from "@/components/DateInputCell.vue";

interface TableRow {
  id: number
  steName: string
  isActual: boolean
  priceEndDate: number
  priceNotNds: number
  nds: number
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
        value: row.isActual,
        onUpdateValue: (value: boolean) => {
          row.isActual = value
          console.log({
            id: row.id,
            isActual: row.isActual,
            priceNotNds: row.priceNotNds,
            nds: row.nds,
            price: row.price,
            priceEndDate: row.priceEndDate
          })
        }
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
    render(row: TableRow) {
      return h(NInputNumber, {
        value: row.priceNotNds,
        min: 0,
        size: 'small',
        showButton: false,
        bordered: false,
        onUpdateValue: (value: number | null) => {
          if (value !== null) {
            row.priceNotNds = value
          }
        }
      })
    }
  },
  {
    key: "nds",
    title: "НДС, %",
    render(row: TableRow) {
      return h(NInputNumber, {
        value: row.nds,
        min: 0,
        max: 100,
        size: 'small',
        showButton: false,
        bordered: false,
        suffix: () => '%',
        onUpdateValue: (value: number | null) => {
          if (value !== null) {
            row.nds  = value
          }
        }
      })
    }
  },
  {
    key: "price",
    title: "Цена, руб. с НДС",
    render(row: TableRow){
      const finalPrice = row.priceNotNds + (row.priceNotNds * row.nds / 100)
      return finalPrice.toFixed(2)
    }
  },
  {
    key: 'fillEndDate',
    title: 'Срок заполнения',
    render(row: TableRow) {
      return h(DateInputCell, {
        value: row.fillEndDate,
        onUpdateValue: (val: string) => {
          row.fillEndDate = val
        }
      })
    }
  }
];

const tableData = ref<TableRow[]>([
  {
    id: 1,
    steName: 'СТЕ №1',
    isActual: true,
    priceEndDate: Date.now(),
    priceNotNds: 1000,
    nds: 20,
    fillEndDate: '2025-12-01',
  },
  {
    id: 2,
    steName: 'СТЕ №2',
    isActual: false,
    priceEndDate: Date.now(),
    priceNotNds: 2000,
    nds: 10,
    fillEndDate: '2025-11-15',
  },
  {
    id: 3,
    steName: 'СТЕ №3',
    isActual: true,
    priceEndDate: Date.now(),
    priceNotNds: 1500,
    nds: 18,
    fillEndDate: '2025-10-20',
  },
  {
    id: 4,
    steName: 'СТЕ №4',
    isActual: false,
    priceEndDate: Date.now(),
    priceNotNds: 500,
    nds: 20,
    fillEndDate: '2025-09-15',
  },
  {
    id: 5,
    steName: 'СТЕ №5',
    isActual: true,
    priceEndDate: Date.now(),
    priceNotNds: 3200,
    nds: 12,
    fillEndDate: '2025-08-01',
  },
  {
    id: 6,
    steName: 'СТЕ №6',
    isActual: true,
    priceEndDate: Date.now(),
    priceNotNds: 2700,
    nds: 15,
    fillEndDate: '2025-12-31',
  },
  {
    id: 7,
    steName: 'СТЕ №7',
    isActual: false,
    priceEndDate: Date.now(),
    priceNotNds: 1800,
    nds: 10,
    fillEndDate: '2025-07-01',
  },
  {
    id: 8,
    steName: 'СТЕ №8',
    isActual: true,
    priceEndDate: Date.now(),
    priceNotNds: 2200,
    nds: 5,
    fillEndDate: '2025-11-01',
  },
  {
    id: 9,
    steName: 'СТЕ №9',
    isActual: false,
    priceEndDate: Date.now(),
    priceNotNds: 800,
    nds: 20,
    fillEndDate: '2025-06-15',
  },
  {
    id: 10,
    steName: 'СТЕ №10',
    isActual: true,
    priceEndDate: Date.now(),
    priceNotNds: 3500,
    nds: 18,
    fillEndDate: '2025-10-10',
  }
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