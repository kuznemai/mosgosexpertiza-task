<script setup lang="ts">
import {ref, h, computed, onMounted, reactive} from 'vue';
import {NSwitch, NDataTable, NDatePicker, NInputNumber} from 'naive-ui';
import DateInputCell from "@/components/DateInputCell.vue";

interface TableRow {
  id: number;
  steName: string;
  isActual: boolean;
  priceEndDate: number;
  priceNotNds: number;
  nds: number;
  fillEndDate: string;
}

const TABLE_HEADERS = [
  {
    key: "steName",
    title: "Наименование СТЕ",
    sorter: (row1: TableRow, row2: TableRow) => {
      return row1.steName.toLowerCase().localeCompare(row2.steName.toLowerCase())
    },
  },
  {
    key: 'isActual',
    title: 'В наличии',
    render(row: TableRow) {
      return h(NSwitch, {
        value: row.isActual,
        onUpdateValue: (value: boolean) => {
          row.isActual = value
          logRowState(row);
        }
      })
    }
  },
  {
    key: "priceEndDate",
    title: "Срок действия предоставленных сведений",
    render(row: TableRow) {
      return h(NDatePicker, {
        value: row.priceEndDate,
        placeholder: "Выберите дату",
        bordered: false,
        size: 'small',
        clearable: true,
        style: {width: '100%'},
        onUpdateValue: (value: number | null) => {
          if (value !== null) {
            row.priceEndDate = value
            logRowState(row);
          }
        }
      })
    }
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
            logRowState(row);
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
            row.nds = value
            logRowState(row);
          }
        }
      })
    }
  },
  {
    key: "price",
    title: "Цена, руб. с НДС",
    render(row: TableRow) {
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
          console.log('val', val)
          row.fillEndDate = val
          logRowState(row);
        }
      })
    }
  }
];

const tableData = ref<TableRow[]>([]);
const currentPage = ref(1);
const pageSize = ref(10);


const sortState = ref<{
  columnKey: string | null;
  order: 'ascend' | 'descend' | null;
}>({
  columnKey: null,
  order: null,
});


const sortedData = computed(() => {
  if (sortState.value.columnKey === 'steName' && sortState.value.order) {
    const sorted = [...tableData.value];
    sorted.sort((a, b) => {
      const nameA = a.steName.toLowerCase();
      const nameB = b.steName.toLowerCase();
      return sortState.value.order === 'ascend'
          ? nameA.localeCompare(nameB)
          : nameB.localeCompare(nameA);
    });
    return sorted;
  }
  return tableData.value;
});


onMounted(async () => {
  try {
    const response = await fetch('https://53d9a3d14d717f4f.mokky.dev/tabledata');
    const data: TableRow[] = await response.json();
    tableData.value = data.map(item => ({
      ...item,
      priceEndDate: typeof item.priceEndDate === 'string' ? Date.parse(item.priceEndDate) : item.priceEndDate,
    }));
  } catch (e) {
    console.error('Ошибка при загрузке данных:', e);
  }
});

const paginatedData = computed(() => {
  const start = (currentPage.value - 1) * pageSize.value;
  const end = start + pageSize.value;
  return sortedData.value.slice(start, end);
});

function logRowState(row: TableRow) {
  const price = +(row.priceNotNds + row.priceNotNds * row.nds / 100).toFixed(2);
  console.log({
    id: row.id,
    isActual: row.isActual,
    priceNotNds: row.priceNotNds,
    nds: row.nds,
    price,
    priceEndDate: row.priceEndDate
  });
}
</script>

<template>
  <div class="custom-table-wrapper">
    <div class="custom-table">
    <n-data-table
        size="large"
        :columns="TABLE_HEADERS"
        :data="paginatedData"
        :bordered="true"
        :single-line="false"
        class="data-table"
    />
    </div>
    <n-pagination v-model:page="currentPage"
                  :page-count="tableData.length/pageSize" class="pagination"/>
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
  background-color: #224c9c;
}

.custom-table :deep(.n-data-table-base-table) {
  overflow: hidden;
}

.custom-table :deep(.n-data-table-th),
.custom-table :deep(.n-data-table-td) {
  border: 1px solid #ccc;
  padding: 12px;
}
.custom-table :deep(.n-data-table-th:hover) {
  background-color: inherit !important;
}
.pagination {
  padding: 15px 0;
}
</style>