<template>
  <div :class="calendar_cell_class">
    <span>{{ dateInfo.date }}</span>
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
});

const calendar_cell_class = computed(() => ({
  "calendar-cell": true,
  "calendar-cell-diluted": !props.dateInfo.isCurrentMonth,
  "calendar-cell-highlighted": props.dateInfo.isToday,
}));
</script>

<style scoped>
.calendar-cell {
  text-align: center;
  padding: 0.1rem;
  /* background-color: #f0f0f0; */
  border-radius: 0.25rem;
  /* 为了弥合与当前日期边框的差距，设置一个粗细相同的边框 */
  border: 3px solid black;
}
.calendar-cell-diluted {
  color: grey;
}
.calendar-cell-highlighted {
  border: 3px solid green;
}
</style>
