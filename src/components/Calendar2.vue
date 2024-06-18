<template>
  <div class="calendar">
    <div class="calendar-setting">
      <label>
        <input type="checkbox" v-model="day_header.if_start_from_monday" />
        Start from Monday
      </label>
      <!-- <label>
        <input type="checkbox" v-model="day_header.if_show_week_number" />
        显示周数
      </label> -->
      <!-- <label>
        <input type="checkbox" v-model="day_header.if_show_day_number" />
        显示天数
      </label> -->
      <!-- <label>
        <input type="checkbox" v-model="day_header.if_show_day_name" />
        显示星期
      </label> -->
      <!-- <label>
        <input type="checkbox" v-model="day_header.if_show_day_date" />
        显示日期
      </label> -->
    </div>
    <div class="calendar-controller">
      <button @click="prevYear">上一年</button>
      <button @click="prevMonth">上一月</button>
      <span>{{ formattedDate }}</span>
      <button @click="focusOnToday">今天</button>
      <button @click="nextMonth">下一月</button>
      <button @click="nextYear">下一年</button>
    </div>
    <div class="calendar-body">
      <div
        class="calendar-cell calender-cell-header"
        v-for="day in day_header.list"
        :key="day"
      >
        {{ day }}
      </div>
      <div class="calendar-cell" v-for="(day, index) in days" :key="index">
        {{ day }}
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, reactive, computed, watch } from "vue";

const date_now = new Date();
const currentDate = ref(new Date());
const day_header_list = ref(["日", "一", "二", "三", "四", "五", "六"]);
let if_start_from_monday = ref(false);
const day_header = reactive({
  if_start_from_monday: false,
  list: ["日", "一", "二", "三", "四", "五", "六"],
});

const day_header_watch = watch(
  // 这里必须使用回调侦听响应式数据的属性
  () => day_header.if_start_from_monday,
  (new_v, old_v) => {
    day_header.list = new_v
      ? ["一", "二", "三", "四", "五", "六", "日"]
      : ["日", "一", "二", "三", "四", "五", "六"];
  }
);

const days = computed(() => {
  const year = currentDate.value.getFullYear();
  const month = currentDate.value.getMonth();

  // 获取当前月份的第一天
  const firstDay = new Date(year, month, 1);
  // 获取该月的天数
  const daysInMonth = new Date(year, month + 1, 0).getDate();
  // 存储要渲染的日期
  const daysArray = [];

  // 插入上月日期
  const last_month_rest =
    (7 + firstDay.getDay() - day_header.if_start_from_monday) % 7;
  for (let i = 0; i < last_month_rest; i++) {
    daysArray.push("");
  }
  // 插入本月日期
  for (let i = 1; i <= daysInMonth; i++) {
    daysArray.push(i);
  }

  return daysArray;
});

const formattedDate = computed(() => {
  const year = currentDate.value.getFullYear();
  const month = currentDate.value.getMonth() + 1;
  return `${year}年${month}月`;
});

const prevYear = () => {
  currentDate.value.setFullYear(currentDate.value.getFullYear() - 1);
  // 触发更新，下同
  currentDate.value = new Date(currentDate.value);
};

const nextYear = () => {
  currentDate.value.setFullYear(currentDate.value.getFullYear() + 1);
  currentDate.value = new Date(currentDate.value);
};

const prevMonth = () => {
  currentDate.value.setMonth(currentDate.value.getMonth() - 1);
  currentDate.value = new Date(currentDate.value);
};

const nextMonth = () => {
  currentDate.value.setMonth(currentDate.value.getMonth() + 1);
  currentDate.value = new Date(currentDate.value);
};

const focusOnToday = () => {
  currentDate.value = date_now;
  currentDate.value = new Date(currentDate.value);
};
</script>

<style>
.calendar {
  max-width: 400px;
  margin: 0 auto;
  background-color: #de75d9;
}

.calendar-setting {
  display: flex;
  justify-content: right;
  align-items: center;
  margin-bottom: 1rem;
}
.calendar-controller {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 0.5rem;
}

.calendar-body {
  display: grid;
  grid-template-columns: repeat(7, 1fr);
  /* gap: 0.5rem; */
}

.calendar-cell {
  text-align: center;
  padding: 0.5rem;
  background-color: #f0f0f0;
  border-radius: 0.25rem;
}

.calender-cell-header {
  font-weight: bold;
}
</style>
