<template>
  <div class="lc-container">
    <input
      type="date"
      name="date_picker"
      id="date_picker"
      v-model="date_picked"
    />
    <p>{{ my_lunar }}</p>
  </div>
</template>

<script setup>
import { ref, reactive } from "vue";

const date_picked = ref(Date());

// 天干地支年
let lunarYearArr = [
  "甲子",
  "乙丑",
  "丙寅",
  "丁卯",
  "戊辰",
  "己巳",
  "庚午",
  "辛未",
  "壬申",
  "癸酉",
  "甲戌",
  "乙亥",
  "丙子",
  "丁丑",
  "戊寅",
  "己卯",
  "庚辰",
  "辛巳",
  "壬午",
  "癸未",
  "甲申",
  "乙酉",
  "丙戌",
  "丁亥",
  "戊子",
  "己丑",
  "庚寅",
  "辛卯",
  "壬辰",
  "癸巳",
  "甲午",
  "乙未",
  "丙申",
  "丁酉",
  "戊戌",
  "己亥",
  "庚子",
  "辛丑",
  "壬寅",
  "癸卯",
  "甲辰",
  "乙巳",
  "丙午",
  "丁未",
  "戊申",
  "己酉",
  "庚戌",
  "辛亥",
  "壬子",
  "癸丑",
  "甲寅",
  "乙卯",
  "丙辰",
  "丁巳",
  "戊午",
  "己未",
  "庚申",
  "辛酉",
  "壬戌",
  "癸亥",
];

// 一些固定数据，例如每月的天数及闰月情况等
let lunarInfo = new Array(
  0xa4b,
  0x5164b,
  0x6a4,
  0x4ac,
  0x12a93,
  0x52a,
  0xa6e,
  0x92e,
  0xc8d7,
  0xc95,
  0xd4a,
  0xda5,
  0x20b55,
  0x56a,
  0x7155b,
  0x25d,
  0x92d,
  0x5192b,
  0xa95,
  0xb4a,
  0x416aa,
  0xad5,
  0x90ab5,
  0x4ba,
  0xa5b,
  0x60a57,
  0x52b,
  0xa93,
  0x40e95
);

function getLunarYearDays(lunarYear) {
  let i,
    sum = 348;
  for (i = 0x8000; i > 0x8; i >>= 1) {
    sum += lunarInfo[lunarYear - 1900] & i ? 1 : 0;
  }
  return sum + getLeapMonthDays(lunarYear);
}

function getLeapMonth(lunarYear) {
  return lunarInfo[lunarYear - 1900] & 0xf;
}

function getLeapMonthDays(lunarYear) {
  if (getLeapMonth(lunarYear)) {
    return lunarInfo[lunarYear - 1900] & 0x10000 ? 30 : 29;
  }
  return 0;
}

function getMonthDays(lunarYear, month) {
  return lunarInfo[lunarYear - 1900] & (0x10000 >> month) ? 30 : 29;
}

// 公历转农历函数
function gregorianToLunar(year, month, day) {
  var baseDate = new Date(1900, 0, 31); //基准日
  var offset = (new Date(year, month - 1, day) - baseDate) / 86400000;

  var temp = 0,
    lunarYear,
    lunarMonth,
    lunarDay;
  for (lunarYear = 1900; lunarYear <= 2100 && offset > 0; lunarYear++) {
    temp = getLunarYearDays(lunarYear);
    offset -= temp;
  }

  if (offset < 0) {
    offset += temp;
    lunarYear--;
  }

  var leapMonth = getLeapMonth(lunarYear);
  var isLeap = false;
  for (lunarMonth = 1; lunarMonth <= 12 && offset > 0; lunarMonth++) {
    // 闰月
    if (leapMonth > 0 && lunarMonth == leapMonth + 1 && !isLeap) {
      --lunarMonth;
      isLeap = true;
      temp = getLeapMonthDays(lunarYear);
    } else {
      temp = getMonthDays(lunarYear, lunarMonth);
    }

    // 已经是闰月，必须跳过,不再计算了
    if (isLeap && lunarMonth == leapMonth + 1) isLeap = false;
    offset -= temp;
  }

  if (offset == 0 && leapMonth > 0 && lunarMonth == leapMonth + 1) {
    if (isLeap) {
      isLeap = false;
    } else {
      isLeap = true;
      --lunarMonth;
    }
  }

  if (offset < 0) {
    offset += temp;
    --lunarMonth;
  }

  lunarDay = offset + 1;

  return {
    lunarYear: lunarYearArr[(lunarYear - 4) % 60], //天干地支年
    lunarMonth: lunarMonth,
    lunarDay: lunarDay,
    isLeap: isLeap,
  };
}

// 使用示例
var gregorianDate = { year: 2024, month: 6, day: 18 };
var lunarDate = gregorianToLunar(
  gregorianDate.year,
  gregorianDate.month,
  gregorianDate.day
);
console.log(lunarDate);

let my_lunar = ref(
  gregorianToLunar(date_picked.year, date_picked.month, date_picked.day)
);
</script>

<!-- style -->
<style scoped>
input[type="date"] {
  border: 2px solid green;
}
</style>
