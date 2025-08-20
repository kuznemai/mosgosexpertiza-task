<script setup lang="ts">
import {ref, h, computed, onMounted, reactive, watch} from "vue";
import {NSwitch, NDataTable, NDatePicker, NInputNumber} from "naive-ui";
import DateInputCell from "@/components/DateInputCell.vue";
import ModalSettingTable from "@/components/ModalSettingTable.vue";
import type {DataTableColumn} from 'naive-ui'

interface TableRow {
  id: number;
  steName: string;
  isActual: boolean;
  priceEndDate: number;
  priceNotNds: number;
  nds: number;
  fillEndDate: string;
  computedPriceWithNds?: number;
}

const TABLE_HEADERS = ref([
  {
    key: "steName",
    title: "Наименование СТЕ",
    sorter: true,
    isShow: true,
    color: "pink",
  },
  {
    key: "isActual",
    title: "В наличии",
    isShow: true,
    style: {backgroundColor: "lightblue"},
    color: "transparent",
    render(row: TableRow) {
      return h(NSwitch, {
        value: row.isActual,
        onUpdateValue: (value: boolean) => {
          row.isActual = value;
          logRowState(row);
        },
      });
    },
  },
  {
    key: "priceEndDate",
    title: "Срок действия предоставленных сведений",
    isShow: true,
    color: "transparent",
    render(row: TableRow) {
      return h(NDatePicker, {
        value: row.priceEndDate,
        placeholder: "Выберите дату",
        bordered: false,
        size: "small",
        clearable: true,
        style: {width: "100%"},
        onUpdateValue: (value: number | null) => {
          if (value !== null) {
            row.priceEndDate = value;
            logRowState(row);
          }
        },
      });
    },
  },
  {
    key: "priceNotNds",
    title: "Цена, руб. без НДС",
    isShow: true,
    color: "transparent",
    render(row: TableRow) {
      return h(NInputNumber, {
        value: row.priceNotNds,
        min: 0,
        size: "small",
        showButton: false,
        bordered: false,
        onUpdateValue: (value: number | null) => {
          if (value !== null) {
            row.priceNotNds = value;
            row.computedPriceWithNds =
                row.priceNotNds + (row.priceNotNds * row.nds) / 100;
            logRowState(row);
          }
        },
      });
    },
  },
  {
    key: "nds",
    title: "НДС, %",
    isShow: true,
    color: "transparent",
    render(row: TableRow) {
      return h(NInputNumber, {
        value: row.nds,
        min: 0,
        max: 100,
        size: "small",
        showButton: false,
        bordered: false,
        suffix: () => "%",
        onUpdateValue: (value: number | null) => {
          if (value !== null) {
            row.nds = value;
            row.computedPriceWithNds =
                row.priceNotNds + (row.priceNotNds * row.nds) / 100;
            logRowState(row);
          }
        },
      });
    },
  },
  {
    key: "price",
    title: "Цена, руб. с НДС",
    isShow: true,
    color: "pink",
    sorter: true,
    render(row: TableRow) {
      return row.computedPriceWithNds?.toFixed(2) ?? "";
    },
  },
  {
    key: "fillEndDate",
    title: "Срок заполнения",
    isShow: true,
    color: "transparent",
    render(row: TableRow) {
      return h(DateInputCell, {
        value: row.fillEndDate,
        onUpdateValue: (val: string) => {
          row.fillEndDate = val;
          logRowState(row);
        },
      });
    },
  },
]);

const tableData = ref<TableRow[]>([]);
const currentPage = ref<number>(1);
const pageSize = ref<number>(10);
const loading = ref<boolean>(false);

onMounted(async () => {

  loading.value = true;
  try {
    const response = await fetch(
        "https://53d9a3d14d717f4f.mokky.dev/tabledata",
    );
    const rawData: TableRow[] = await response.json();

    tableData.value = rawData.map((item) => {
      const row = reactive({
        ...item,
        priceEndDate:
            typeof item.priceEndDate === "string"
                ? Date.parse(item.priceEndDate)
                : item.priceEndDate,
        computedPriceWithNds:
            item.priceNotNds + (item.priceNotNds * item.nds) / 100,
      });

      return row;
    });
  } catch (e) {
    console.error("Ошибка при загрузке данных:", e);
  } finally {
    loading.value = false;
  }
  settingsStorage.value = JSON.parse(localStorage.getItem('tableSettings'));

  TABLE_HEADERS.value.map(elem => {
    settingsStorage.value .forEach(el => {
      if (elem.key === el.key) {
        elem.isShow = el.isShow;
        elem.color = el.color;
      }
    });
  });
});

function handleSorterChange(sorter) {
  const sorted = [...tableData.value];
  console.log("sorter.columnKey", sorter);
  if (sorter.columnKey === "steName") {
    sorted.sort((a, b) => {
      const nameA = a.steName.toLowerCase();
      const nameB = b.steName.toLowerCase();
      if (sorter.order === "ascend") {
        return nameA.localeCompare(nameB);
      } else if (sorter.order === "descend") {
        return nameB.localeCompare(nameA);
      } else {
        return 0;
      }
    });
    tableData.value = sorted;
  }
  if (sorter.columnKey === "price") {
    sorted.sort((a, b) => {
      return sorter.order === "ascend"
          ? a.computedPriceWithNds! - b.computedPriceWithNds!
          : b.computedPriceWithNds! - a.computedPriceWithNds!;
    });
    tableData.value = sorted;
  }
}

const paginatedData = computed<TableRow[]>(() => {
  const start = (currentPage.value - 1) * pageSize.value;
  const end = start + pageSize.value;
  return tableData.value.slice(start, end);
});

function logRowState(row: TableRow) {
  console.log({
    id: row.id,
    isActual: row.isActual,
    priceNotNds: row.priceNotNds,
    nds: row.nds,
    priceEndDate: row.priceEndDate,
    computedPriceWithNds: row.computedPriceWithNds,
  });
}

const pageSizes = computed(() => {
  const total = tableData.value.length;
  const sizes: number[] = [];
  const baseSizes = [10, 20, 30, 40, 50, 60, 70, 80, 90, 100];

  for (const size of baseSizes) {
    if (size < total) {
      sizes.push(size);
    }
  }
  sizes.push(total);

  return sizes;
});

const showModal = ref(false);

function showSettings() {
  showModal.value = true;
}

const tableDataSettings = computed(() =>
    TABLE_HEADERS.value.map((item) => {
      return {
        key: item.key,
        title: item.title,
        isShow: item.isShow,
        color: item.color,
      };
    }),
);

const settingsStorage = ref([])
function changeSettings(payload: { key: string, isShow?: boolean, color?: string }) {
  console.log(payload)
  const existingSetting = settingsStorage.value.find((item) => item.key === payload.key);
  console.log(existingSetting)

  if(existingSetting) {
    if(payload.isShow!== undefined){
      existingSetting.isShow = payload.isShow;
    }
    if(payload.color!== undefined){
      existingSetting.color = payload.color;
    }
  }else{
    settingsStorage.value.push(payload);
  }


  TABLE_HEADERS.value.forEach((item) => {
    if (item.key === payload.key) {

      if(payload.isShow !== undefined) {
        item.isShow = payload.isShow;
      }
      if(payload.color){
        item.color = payload.color;
      }
    }
  })
  localStorage.setItem('tableSettings', JSON.stringify(settingsStorage.value));
}

console.log('settingsStorage',settingsStorage.value);
const columns = computed<DataTableColumn<TableRow>[]>(() =>
    TABLE_HEADERS.value
        .filter(c => c.isShow)
        .map((c) => {
          const baseRender = c.render as DataTableColumn<TableRow>['render'] | undefined
          return {
            ...c,
            render: baseRender ?? ((row: TableRow) => (row as any)[c.key]),
            cellProps: () => ({
              style: {
                backgroundColor: c.color,
                padding: '8px'
              }
            })
          } as DataTableColumn<TableRow>
        })
)
</script>

<template>
  <div class="custom-table-wrapper">
    <n-icon size="30" @click="showSettings">
      <svg
          version="1.1"
          xmlns="http://www.w3.org/2000/svg"
          xmlns:xlink="http://www.w3.org/1999/xlink"
          x="0px"
          y="0px"
          viewBox="0 0 512 512"
          enable-background="new 0 0 512 512"
          xml:space="preserve"
      >
        <g>
          <path
              d="M413.967,276.8c1.06-6.235,1.06-13.518,1.06-20.8s-1.06-13.518-1.06-20.8l44.667-34.318
		c4.26-3.118,5.319-8.317,2.13-13.518L418.215,115.6c-2.129-4.164-8.507-6.235-12.767-4.164l-53.186,20.801
		c-10.638-8.318-23.394-15.601-36.16-20.801l-7.448-55.117c-1.06-4.154-5.319-8.318-10.638-8.318h-85.098
		c-5.318,0-9.577,4.164-10.637,8.318l-8.508,55.117c-12.767,5.2-24.464,12.482-36.171,20.801l-53.186-20.801
		c-5.319-2.071-10.638,0-12.767,4.164l-42.549,71.765c-2.119,4.153-1.061,10.399,2.129,13.518L96.97,235.2
		c0,7.282-1.06,13.518-1.06,20.8s1.06,13.518,1.06,20.8l-44.668,34.318c-4.26,3.118-5.318,8.317-2.13,13.518L92.721,396.4
		c2.13,4.164,8.508,6.235,12.767,4.164l53.187-20.801c10.637,8.318,23.394,15.601,36.16,20.801l8.508,55.117
		c1.069,5.2,5.318,8.318,10.637,8.318h85.098c5.319,0,9.578-4.164,10.638-8.318l8.518-55.117c12.757-5.2,24.464-12.482,36.16-20.801
		l53.187,20.801c5.318,2.071,10.637,0,12.767-4.164l42.549-71.765c2.129-4.153,1.06-10.399-2.13-13.518L413.967,276.8z
		 M255.468,328.8c-41.489,0-74.46-32.235-74.46-72.8s32.971-72.8,74.46-72.8s74.461,32.235,74.461,72.8S296.957,328.8,255.468,328.8
		z"
          ></path>
        </g>
      </svg>
    </n-icon>
    <n-modal v-model:show="showModal" preset="dialog" title="Настройки">
      <ModalSettingTable
          :table-data-settings="tableDataSettings"
          @update:settings="changeSettings"
      ></ModalSettingTable>
      <template #action>
        <n-button @click="showModal = false">Закрыть</n-button>
      </template>
    </n-modal>

    <n-spin :show="loading" description="Загрузка...">
      <div style="overflow-x: auto">
        <div class="custom-table">
          <n-data-table
              size="large"
              :columns="columns"
              :data="paginatedData"
              :bordered="true"
              :single-line="false"
              class="data-table"
              @update:sorter="handleSorterChange"
          />
        </div>
      </div>
    </n-spin>

    <n-pagination
        v-model:page="currentPage"
        v-model:page-size="pageSize"
        :page-count="Math.ceil(tableData.length / pageSize)"
        show-size-picker
        :page-sizes="pageSizes"
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
  font-family: "Segoe UI", sans-serif;
  font-size: 14px;
}

.custom-table :deep(.n-data-table-th) {
  background-color: #5483e6;
  font-weight: 600;
  color: #ffffff;
  text-transform: uppercase;
}

.custom-table :deep(.n-data-table-th:hover) {
  background-color: #224c9c !important;
}

.custom-table :deep(.n-data-table-td) {
  padding: 8px;
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
  filter: brightness(95%);
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

.modal-columns {
  padding: 8px 0;
}

.modal-column {
  font-size: 16px;
  font-weight: 500;
  margin: 0;
  padding: 8px;
  border-bottom: 1px solid #ddd;
}

.modal-column span {
  margin-right: 10px;
}

.n-data-table-td input {
  background-color: transparent !important;
}

@media (max-width: 1024px) {
  .custom-table-wrapper {
    padding: 16px;
  }

  .custom-table :deep(.n-data-table) {
    font-size: 13px;
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
