<script setup lang="ts">
import {nextTick, ref, watch} from 'vue'
import { NInput } from 'naive-ui'

const props = defineProps<{ value: string }>()
const emit = defineEmits<{ (e: 'updateValue', val: string): void }>()

const editing = ref(false)
const inputValue = ref('')

function isoToLocal(iso: string): string {
  if (!iso) return ''
  const [year, month, day] = iso.split('-')
  return `${day}.${month}.${year}`
}

function localToIso(local: string): string | null {
  const parts = local.split('.')
  if (parts.length !== 3) return null
  const [day, month, year] = parts
  if (day.length !== 2 || month.length !== 2 || year.length !== 4) return null
  return `${year}-${month}-${day}`
}

function isValidDateString(dateStr: string): boolean {
  const regex = /^\d{2}\.\d{2}\.\d{4}$/
  if (!regex.test(dateStr)) return false
  const [day, month, year] = dateStr.split('.').map(Number)
  const date = new Date(year, month - 1, day)
  return (
      date.getFullYear() === year &&
      date.getMonth() === month - 1 &&
      date.getDate() === day
  )
}

watch(() => props.value, (newVal) => {
  inputValue.value = isoToLocal(newVal)
}, { immediate: true })

function onClickCell() {
  editing.value = true
  nextTick(() => {
    const input = document.querySelector('input')
    input?.focus()
  })
}

function handleBlur() {
  const trimmed = inputValue.value.trim()

  if (trimmed === '') {
    emit('updateValue', '')
  } else if (isValidDateString(trimmed)) {
    const iso = localToIso(trimmed)
    if (iso) {
      emit('updateValue', iso)
    } else {
      inputValue.value = isoToLocal(props.value)
    }
  } else {
    alert('Некорректная дата. Используйте формат ДД.ММ.ГГГГ')
    inputValue.value = isoToLocal(props.value)
  }

  editing.value = false
}


</script>

<template>
  <div @click="onClickCell" class="date-cell" >
    <template v-if="editing">
      <n-input
          v-model:value="inputValue"
          size="small"
          placeholder="ДД.ММ.ГГГГ"
          @blur="handleBlur"
          class="date-input"
      />

    </template>
    <template v-else>
      {{ inputValue || '—' }}
    </template>
  </div>
</template>
<style scoped>
.date-cell {
  min-width: 100px;
  line-height: 24px;
  overflow: hidden;
}

.date-cell input {
  width: 100%;
  height: 100%;
  padding: 0;
  margin: 0;
  border: none;
  box-shadow: none;
  background: transparent;
  font-size: 14px;
  line-height: 24px;
  box-sizing: border-box;
}
</style>
