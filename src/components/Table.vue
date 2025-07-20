<script setup lang="ts">
import {ref, h, computed, onMounted, reactive, watch} from 'vue';
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
    sorter: true
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
const loading = ref(false);

onMounted(async () => {
  loading.value = true;
  try {
    const response = await fetch('https://53d9a3d14d717f4f.mokky.dev/tabledata');
    const data: TableRow[] = await response.json();
    tableData.value = data.map(item => ({
      ...item,
      priceEndDate: typeof item.priceEndDate === 'string' ? Date.parse(item.priceEndDate) : item.priceEndDate,
    }));
  } catch (e) {
    console.error('Ошибка при загрузке данных:', e);
  } finally {
    loading.value = false;
  }
});


function handleSorterChange(sorter) {
  const sorted = [...tableData.value];
  if (sorter.columnKey === 'steName') {
    sorted.sort((a, b) => {
      const nameA = a.steName.toLowerCase();
      const nameB = b.steName.toLowerCase();
      if (sorter.order === 'ascend') {
        return nameA.localeCompare(nameB);
      } else if (sorter.order === 'descend') {
        return nameB.localeCompare(nameA);
      } else {
        return 0;
      }
    });
    tableData.value = sorted;
  }
}

  const paginatedData = computed(() => {
    const start = (currentPage.value - 1) * pageSize.value;
    const end = start + pageSize.value;
    return tableData.value.slice(start, end);
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
    <n-spin :show="loading" description="Загрузка...">
      <div style="overflow-x: auto;">
        <div class="custom-table">
          <n-data-table
              size="large"
              :columns="TABLE_HEADERS"
              :data="paginatedData"
              :bordered="true"
              :single-line="false"
              @update:sorter="handleSorterChange"
              class="data-table"
          />
        </div>
      </div>
    </n-spin>
    <n-pagination
        v-model:page="currentPage"
        :page-count="Math.ceil(tableData.length / pageSize)"
        class="pagination"
    />
  </div>
</template>

<style scoped>
.custom-table-wrapper {
  padding: 24px;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.06);
  max-width: 100%;
  overflow-x: auto;
}

.custom-table {
  min-width: 800px;
}
.custom-table :deep(.n-data-table) {
  width: 100%;
  font-family: 'Segoe UI', sans-serif;
  font-size: 14px;
}

.custom-table :deep(.n-data-table-th) {
  background-color: #5483e6;
  font-weight: 600;
  color: #ffffff;
  text-transform: uppercase;
}

.custom-table :deep(.n-data-table-td),
.custom-table :deep(.n-data-table-th) {
  padding: 12px;
  border: 1px solid #ccc;
}

.custom-table :deep(.n-data-table-sorter) svg {
  width: 18px;
  height: 18px;
  fill: #ffffff;
  color: #ffffff;
}

.custom-table :deep(.n-data-table-sorter:hover) svg {
  fill: #ffd700;
  color: #ffd700;
}

.custom-table :deep(.n-data-table-tr:hover) {
  background-color: #224c9c;
  color: #fff;
}

.custom-table :deep(.n-data-table-tr:hover .n-input),
.custom-table :deep(.n-data-table-tr:hover .n-date-picker),
.custom-table :deep(.n-data-table-tr:hover .n-input-number) {
  background-color: transparent;
  color: inherit;
}

.custom-table :deep(.n-data-table-tr:hover input),
.custom-table :deep(.n-data-table-tr:hover .n-base-suffix),
.custom-table :deep(.n-data-table-tr:hover .n-input__suffix),
.custom-table :deep(.n-data-table-tr:hover .n-input__prefix) {
  background-color: transparent;
  color: inherit;
  fill: inherit;
}

.pagination {
  padding: 15px 0;
  display: flex;
  justify-content: center;
}

@media (max-width: 1024px) {
  .custom-table-wrapper {
    padding: 16px;
  }

  .custom-table :deep(.n-data-table) {
    font-size: 13px;
  }

  .custom-table :deep(.n-data-table-th),
  .custom-table :deep(.n-data-table-td) {
    padding: 8px;
  }
}

@media (max-width: 600px) {
  .custom-table-wrapper {
    padding: 12px;
  }

  .custom-table {
    min-width: 600px;
  }

  .pagination {
    flex-direction: column;
    align-items: center;
  }
}
</style>