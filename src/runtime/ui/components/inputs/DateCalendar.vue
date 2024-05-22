<script setup lang="ts">
import { upperFirst } from 'scule'
import { addDays, addMonths, endOfMonth, intlFormat, startOfMonth, subDays, subMonths } from 'date-fns'
import Separator from '../elements/Separator.vue'
import Card from '../elements/Card.vue'
import { type Ref, computed, ref, useNuxtApp, watch } from '#imports'

const props = defineProps<{
  selected: string[]
}>()
const emit = defineEmits(['select'])
const app = useNuxtApp()
const locale = app.$locale as Ref<string>

function formatDate(date: Date) {
  return new Date(date.getTime() - (date.getTimezoneOffset() * 60000))
    .toISOString().split('T')[0]
}

const calendarDate = ref(new Date())
const displayMonth = computed(() => {
  return upperFirst(
    intlFormat(calendarDate.value, {
      year: 'numeric',
      month: 'long',
    }, { locale: locale.value }),
  )
})

const weekDays = (() => {
  const { format } = new Intl.DateTimeFormat(locale.value, { weekday: 'short' })
  return [...Array(7).keys()]
    .map(day => format(new Date(Date.UTC(2021, 5, day))))
})()

const days = computed(() => {
  const days = []
  const today = formatDate(new Date())
  let firstDay = subDays(startOfMonth(calendarDate.value), 1)
  while (firstDay.getDay() !== 1)
    firstDay = subDays(firstDay, 1)
  let lastDay = addDays(endOfMonth(calendarDate.value), 1)
  while (lastDay.getDay() !== 6)
    lastDay = addDays(lastDay, 1)
  lastDay = addDays(lastDay, 1)
  for (let date = firstDay; date <= lastDay; date = addDays(date, 1)) {
    const formated = formatDate(date)
    days.push({
      label: date.getDate(),
      formated,
      isToday: formated === today,
      inCalendarMonth: date.getMonth() === calendarDate.value.getMonth()
      && date.getFullYear() === calendarDate.value.getFullYear(),
    })
  }
  return days
})

let skipUpdate = false
function setAuto() {
  if (skipUpdate) {
    skipUpdate = false
    return
  }
  calendarDate.value = new Date(props.selected[0])
}
watch(() => props.selected, setAuto, { immediate: true })

function select(date: string) {
  emit('select', date)
  skipUpdate = true
}
</script>

<template>
  <Card>
    <!-- month -->
    <div flex flex-row items-center>
      <button
        color="default" class="clickable px-3 py-2"
        @click="calendarDate = subMonths(calendarDate, 1)"
      >
        <Icon name="material-symbols:chevron-left-rounded" />
      </button>
      <div flex-1 text-center>
        {{ displayMonth }}
      </div>
      <button
        color="default" class="clickable px-3 py-2"
        @click="calendarDate = addMonths(calendarDate, 1)"
      >
        <Icon name="material-symbols:chevron-right-rounded" />
      </button>
    </div>
    <Separator />

    <!-- weekdays -->
    <div class="flex flex-wrap p-2">
      <div
        v-for="wd in weekDays" :key="wd"
        class="flex-1 text-center text-sm uppercase text-faint"
      >
        {{ wd }}
      </div>
    </div>

    <!-- days -->
    <div class="flex flex-wrap p-2">
      <div
        v-for="day in days" :key="day.formated"
        class="flex-1/7 p-1px text-center"
      >
        <div
          class="clickable rounded p-1 ring-neutral-500"
          :color="selected.includes(day.formated) ? 'contrast' : 'default'"
          :class="{ 'text-faint': !day.inCalendarMonth, 'ring-1': (day.isToday && !selected.includes(day.formated)) }"
          @click="select(day.formated)"
        >
          {{ day.label }}
        </div>
      </div>
    </div>
  </Card>
</template>