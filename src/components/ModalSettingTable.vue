<script setup lang="ts">
import { NDataTable, NSwitch } from "naive-ui";
import { h } from "vue";

interface Props {
  tableDataSettings: {
    key: string;
    title: string;
    isShow: boolean;
    color: string;
  }[];
}
const props = defineProps<Props>();
const emit = defineEmits(["update:isShow", "update:color"]);
const TABLE_HEADERS = [
  { key: "title", title: "Название колонки" },
  {
    key: "isShow",
    title: "Показывать/Не показывать",
    render(row: Props["tableDataSettings"][number]) {
      return h(NSwitch, {
        value: row.isShow,
        "onUpdate:value": (value: boolean) => {
          row.isShow = value;
          emit("update:isShow", row.key, value);
        },
      });
    },
  },
  {
    key: "color",
    title: "Цвет",
    render(row: Props["tableDataSettings"][number]) {
      return h("input", {
        type: "color",
        value: row.color,
        style: {
          width: "40px",
          height: "24px",
          border: "none",
          background: "transparent",
          cursor: "pointer",
        },
        onInput: (e: Event) => {
          const target = e.target as HTMLInputElement;
          row.color = target.value;
          emit("update:color", row.key, target.value);
        },
      });
    },
  },
];
</script>

<template>
  <n-data-table
    size="large"
    :columns="TABLE_HEADERS"
    :data="tableDataSettings"
    :bordered="true"
    :single-line="false"
    class="data-table"
  />
  <!--  <div v-for="column in tableSettings" :key="column.key" class="modal-columns">-->
  <!--    <div class="modal-column">-->
  <!--      <span>{{ column.title }}</span>-->
  <!--      <n-switch size="small" />-->
  <!--      <div>-->
  <!--        <input type="color" value="#e66465" />-->
  <!--        <label for="head">Выбрать цвет</label>-->
  <!--      </div>-->
  <!--    </div>-->
  <!--  </div>-->
</template>

<style scoped></style>
