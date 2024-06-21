<template>
  <div :class="calendar_cell_class">
    <span>{{ dateInfo.date.getDate() }}</span>
    <!-- <div v-for="(record, index) in records" :key="index"></div> -->
  </div>
</template>

<script setup>
import { ref, reactive, computed } from "vue";
const props = defineProps({
  dateInfo: {
    type: Object,
    required: true,
  },
  selected_date: {
    type: Date,
  },
});

const calendar_cell_class = computed(() => ({
  "calendar-cell": true,
  "calendar-cell-diluted": !props.dateInfo.isCurrentMonth,
  // "calendar-cell-highlighted": props.dateInfo.isToday,
  "calendar-cell-selected": props.selected_date == props.dateInfo.date,
}));
</script>

<style scoped>
.calendar-cell {
  display: flex;
  justify-content: center;
  align-items: center;
  text-align: center;
  padding: 0.1rem;
  height: 1.5rem;
  border-radius: 0.25rem;
  box-sizing: border-box;
}
.calendar-cell-diluted {
  color: grey;
}
.calendar-cell-selected {
  border: 3px solid whitesmoke;
  /* font-size: large; */
}
</style>
