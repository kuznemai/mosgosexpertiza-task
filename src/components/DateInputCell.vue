<template>
  <n-date-picker
      :value="dateValue"
      size="small"
      placeholder="Выберите дату"
      :bordered="false"
      clearable
      style="width: 100%"
      @update:value="handleUpdate"
  />
</template>

<script setup lang="ts">
import {ref, watch} from 'vue';

const props = defineProps<{
  value: string;
  onUpdateValue: (val: string) => void;
}>();

const dateValue = ref<string | null>(props.value);

watch(() => props.value, (newVal) => {
  dateValue.value = newVal;
});

function handleUpdate(val: number | null) {
  if (val !== null) {
    const iso = new Date(val).toISOString().split('T')[0];
    props.onUpdateValue(iso);
  }
}
</script>
