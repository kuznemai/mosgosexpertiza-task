<script setup lang="ts">
import { NDataTable, NSwitch } from "naive-ui";

interface Props {
  tableDataSettings: {
    key: string;
    title: string;
    isShow: boolean;
    color: string;
  }[];
}
const props = defineProps<Props>();
const TABLE_HEADERS = [
  { key: "title", title: "Название колонки" },
  { key: "isShow", title: "Показывать/Не показывать" },
  { key: "color", title: "Цвет" },
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
