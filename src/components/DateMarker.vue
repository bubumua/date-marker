<template>
  <div class="datemarker">
    <div class="calendar">
      <div class="calendar-setting">
        <label>
          <input type="checkbox" v-model="day_header.if_start_from_monday" />
          Start from Monday
        </label>
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
        <div
          v-for="(dateInfo, index) in dateInfos"
          :key="index"
          :dateInfo="dateInfo"
          :selected_date="selected_date"
          class="calendar-cell"
          :class="{
            'calendar-cell-diluted': !dateInfo.isCurrentMonth,
            'calendar-cell-highlighted': dateInfo.isToday,
            'calendar-cell-selected': isSelectedDate(dateInfo.date),
          }"
          :style="calendar_cell_activity_highlight(dateInfo)"
          @click="select_date(dateInfo.date)"
        >
          <span>{{ dateInfo.date.getDate() }}</span>
        </div>
      </div>
    </div>
    <div class="tag_area">
      <ul class="tags_container">
        <li
          v-for="(tag, index) in c_tags"
          :key="index"
          class="tag custom_tag"
          :class="{ 'selected-tag': tag.isSelected }"
          :style="{ backgroundColor: tag.color }"
          draggable="true"
          @dragstart="dragStart(index)"
          @dragover.prevent
          @drop="drop(index)"
          @dragenter="dragEnter(index)"
          @click="selectTag(tag)"
        >
          <button
            class="delete-btn"
            v-show="isEditing"
            @click.stop="deleteTag(index)"
          >
            X
          </button>
          {{ tag.name }}
        </li>
        <li
          class="tag plus-tag"
          v-show="isEditing"
          style="background-color: yellow"
          @click="showAddTagModal"
        >
          +add
        </li>
      </ul>
      <!-- </div> -->
      <!-- 编辑按钮 -->
      <div class="edit">
        <label class="col_checkbox">
          <input type="checkbox" v-model="isEditing" />
          Edit
          <!-- {{ isEditing ? "true" : "false" }} -->
        </label>
      </div>
      <!-- Modal for adding tag -->
      <div v-if="showModal" class="modal">
        <div class="modal-content">
          <h3>Add a new tag</h3>
          <input v-model="newTag.name" placeholder="Enter tag name" />
          <button @click="addTag(newTag.name, selected_color)" v-if="isAdding">
            Add
          </button>
          <button @click="closeAddTagModal">Cancel</button>
        </div>
        <div class="sepan">
          <label
            v-for="color in COLORS"
            :for="color.name"
            :key="color.name"
            :class="[
              'color',
              { 'color-selected': selected_color.name === color.name },
            ]"
            :style="{ backgroundColor: color.rgb }"
          >
            <input
              type="radio"
              v-model="selected_color"
              :value="color"
              :id="color.name"
            />
          </label>
          <!-- {{ color.name }} -->
        </div>
      </div>
    </div>
    <div class="operation-area">
      <button class="btn-operation btn-mark" @click="addActivityRecord">
        Mark
      </button>
      <button class="btn-operation btn-erase" @click="deleteActivityRecord">
        Erase
      </button>
    </div>
  </div>
</template>

<script setup>
import { ref, reactive, computed, watch } from "vue";

// 今日日期
const full_date_now = new Date();
// 日历上的当前日期
const currentDate = ref(new Date());
// 当前选择的日期
const selected_date = ref(new Date());
// 日历表头
const day_header = reactive({
  if_start_from_monday: false,
  list: ["日", "一", "二", "三", "四", "五", "六"],
});
// 日历显示方式改变侦听器
const day_header_watch = watch(
  // 这里必须使用回调侦听响应式数据的属性
  () => day_header.if_start_from_monday,
  (new_v, old_v) => {
    day_header.list = new_v
      ? ["一", "二", "三", "四", "五", "六", "日"]
      : ["日", "一", "二", "三", "四", "五", "六"];
  }
);
// 定义颜色
const COLORS = [
  { name: "red", rgb: "#ff0000" },
  { name: "pink", rgb: "#f5429e" },
  { name: "orange", rgb: "#f58b42" },
  { name: "apricot", rgb: "#f5a572" },
  { name: "peach", rgb: "#f5a892" },
  { name: "gold", rgb: "#f5b142" },
  { name: "yell", rgb: "#d3d301" },
  { name: "olive", rgb: "#a1cb42" },
  { name: "green", rgb: "#008000" },
  { name: "blue", rgb: "#4385f5" },
  { name: "navy", rgb: "#4259cb" },
  { name: "indigo", rgb: "#6342f5" },
  { name: "violet", rgb: "#a142f5" },
  { name: "purple", rgb: "#9259cb" },
  { name: "magenta", rgb: "#cb42ab" },
  { name: "lavender", rgb: "#e0b3ff" },
  { name: "charcoal", rgb: "#4a4a4a" },
  { name: "maroon", rgb: "#5c3d42" },
];
// 编辑状态
const isEditing = ref(Boolean(false));
const isAdding = ref(Boolean(false));
const showModal = ref(Boolean(false));
// tag拖拽
const draggedTagIndex = ref(-1);
const selected_color = ref(COLORS[0]);
// 用户tags
const tags = reactive([
  {
    name: "看看你的",
    color: COLORS[0].rgb,
    isSelected: false,
  },
]);
// activity records
// {
//   activity_name: {
//     color: String,
//     order: Number,
//     records: [ISODate],
//    } ,
// }
const activityRecord = reactive({});
// 处理过选中属性的tags
const c_tags = computed(() => {
  const firstTrueObject = tags.find((obj) => obj.isSelected === true);
  if (firstTrueObject) {
    // 如果找到了，将其他对象的 isSelected 属性设为 false
    tags.forEach((obj) => {
      if (obj !== firstTrueObject) {
        obj.isSelected = false;
      }
    });
  } else {
    // 如果没有找到，设置第一个对象的 isSelected 属性为 true
    if (tags.length > 0) {
      tags.forEach((obj) => {
        obj.isSelected = false;
      });
      tags[0].isSelected = true;
    }
  }
  return tags;
});
// 当前选择的tag
const selected_tag = ref(tags.isEmpty ? {} : tags[0]);
// 暂存新tag
let newTag = reactive({
  name: "new tag",
  color: COLORS[0].rgb,
});

// 在日期层面上比较两个Date
const compareDateOnDate = (date1, date2) => {
  const d1 = new Date(date1);
  const d2 = new Date(date2);
  d1.setHours(0, 0, 0, 0);
  d2.setHours(0, 0, 0, 0);
  if (d1 < d2) {
    return -1;
  } else if (d1 > d2) {
    return 1;
  } else {
    return 0;
  }
};

const dateInfos = computed(() => {
  // 存储要渲染的日期
  const datesArray = [];
  // 获取当前日期的年份和月份
  const year = currentDate.value.getFullYear();
  const month = currentDate.value.getMonth();

  // 插入前月日期
  // 获取当前月份的第一天
  const firstDay = new Date(year, month, 1);
  // 计算当月缩进
  const last_month_rest_count =
    (7 + firstDay.getDay() - day_header.if_start_from_monday) % 7;
  // 前一月的天数
  const daysInLastMonth = new Date(year, month, 0).getDate();
  const calendar_start_date = daysInLastMonth - last_month_rest_count + 1;
  for (let i = calendar_start_date; i <= daysInLastMonth; i++) {
    datesArray.push({
      date: new Date(year, month - 1, i),
      isToday: false,
      isCurrentMonth: false,
      activity: null,
    });
  }
  // 插入本月日期
  // 获取本月的天数（日期参数0表示获取上个月的最后一天的日期）
  const daysInMonth = new Date(year, month + 1, 0).getDate();
  const date_today = full_date_now.getDate();
  for (let i = 1; i <= daysInMonth; i++) {
    datesArray.push({
      date: new Date(year, month, i),
      isToday:
        currentDate.value.getFullYear() == full_date_now.getFullYear() &&
        currentDate.value.getMonth() == full_date_now.getMonth() &&
        date_today == i,
      isCurrentMonth: true,
      activity: null,
    });
  }

  // 插入下月日期
  const last_cur_sum_len = datesArray.length;
  const calendar_end_date = 42 - last_cur_sum_len;
  for (let i = 1; i <= calendar_end_date; i++) {
    datesArray.push({
      date: new Date(year, month + 1, i),
      isToday: false,
      isCurrentMonth: false,
      activity: null,
    });
  }
  // 计算日期上是否有记录
  if (activityRecord[selected_tag.value.name]) {
    let records = activityRecord[selected_tag.value.name].records;
    records.reverse();
    // 利用记录的时间有序性，从记录的尾部向前查，即从新到旧遍历。
    for (const recordDate of records) {
      const recordDate_month = recordDate.getMonth();
      const recordDate_date = recordDate.getDate();
      // 若记录在当前日历范围内，则标记对应日期，直至某个记录不在范围内，结束遍历。
      if (
        !(
          compareDateOnDate(recordDate, datesArray[0].date) < 0 ||
          compareDateOnDate(recordDate, datesArray.at(-1).date) > 0
        )
      ) {
        if (recordDate_month == month) {
          datesArray.at(last_month_rest_count + recordDate_date - 1).activity =
            selected_tag.value;
        } else if (
          recordDate_month == new Date(year, month - 1, 1).getMonth()
        ) {
          datesArray.at(recordDate_date - calendar_start_date).activity =
            selected_tag.value;
        } else {
          datesArray.at(last_cur_sum_len + recordDate_date - 1).activity =
            selected_tag.value;
        }
      } else {
        break;
      }
    }
  }
  return datesArray;
});

const select_date = (date) => {
  selected_date.value = date;
};

const isSelectedDate = (date) => {
  return (
    date.getFullYear() == selected_date.value.getFullYear() &&
    date.getMonth() == selected_date.value.getMonth() &&
    date.getDate() == selected_date.value.getDate()
  );
};

const isSameDate = (date1, date2) => {
  return (
    date1.getFullYear() === date2.getFullYear() &&
    date1.getMonth() === date2.getMonth() &&
    date1.getDate() === date2.getDate()
  );
};

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
  currentDate.value = full_date_now;
  currentDate.value = new Date(currentDate.value);
};

// 添加tag菜单
const showAddTagModal = () => {
  isAdding.value = true;
  showModal.value = true;
};
const closeAddTagModal = () => {
  showModal.value = false;
  isAdding.value = false;
};
const addTag = (name, color = COLORS[0]) => {
  if (name !== "") {
    newTag = {
      name: name,
      color: color.rgb,
      isSelected: false,
    };
    tags.push(newTag);
    closeAddTagModal();
  }
};

// tag 拖拽相关
const dragStart = (index) => {
  draggedTagIndex.value = index;
};
const dragEnter = (index) => {
  const draggedOverItemIndex = index;
  const draggedItem = tags[draggedTagIndex.value];
  tags.splice(draggedTagIndex.value, 1);
  tags.splice(draggedOverItemIndex, 0, draggedItem);
  draggedTagIndex.value = draggedOverItemIndex;
};
const drop = () => {
  draggedTagIndex.value = null;
};

// 选中tag
const selectTag = (tag) => {
  for (let t of tags) {
    if (t) t.isSelected = false;
  }
  tag.isSelected = true;
  selected_tag.value = tag;
};

// 删除tag
const deleteTag = (index) => {
  tags.splice(index, 1);
};

// 增加记录
const addActivityRecord = () => {
  const tagIndex = tags.findIndex(
    (item) => item.name === selected_tag.value.name
  );
  if (tagIndex < 0) {
    console.log("error: findIndex in addActivityRecord < 0");
    return;
  }
  // 若已有该活动的记录，则追加并排序
  if (selected_tag.value.name in activityRecord) {
    activityRecord[selected_tag.value.name].records.push(selected_date.value);
    activityRecord[selected_tag.value.name].records.sort((a, b) => a - b);
  }
  // 若没有该活动的记录，则新建
  else {
    activityRecord[selected_tag.value.name] = {
      color: selected_tag.value.color,
      order: tagIndex,
      records: [],
    };
    activityRecord[selected_tag.value.name].records.push(selected_date.value);
  }
  // console.log(`activityRecord: ${activityRecord}`);
};
// 删除记录
const deleteActivityRecord = () => {
  const tagIndex = tags.findIndex(
    (item) => item.name === selected_tag.value.name
  );
  if (tagIndex < 0) {
    console.log("error: findIndex in addActivityRecord < 0");
    return;
  }
  // 若有该活动的记录，则删除最新的记录
  if (selected_tag.value.name in activityRecord) {
    activityRecord[selected_tag.value.name].records.reverse();
    const duplicated_index = activityRecord[
      selected_tag.value.name
    ].records.findIndex((dt) => isSameDate(dt, selected_date.value));
    if (duplicated_index >= 0) {
      activityRecord[selected_tag.value.name].records.splice(
        duplicated_index,
        1
      );
    }
    activityRecord[selected_tag.value.name].records.reverse();
  }
  console.log(activityRecord);
};

// 根据记录设置日期背景色
const calendar_cell_activity_highlight = (dateInfo) => {
  if (dateInfo.activity) {
    return { backgroundColor: dateInfo.activity.color };
  } else {
    return {};
  }
};
</script>

<style scoped>
.datemarker {
  display: flex;
  flex-flow: column nowrap;
  justify-content: start;
  align-items: center;
  --backgroundColor: black;
  --highlightSelected: whitesmoke;
  --textColor: white;
}
/* calendar begin */
.calendar {
  width: 100%;
  margin: 0;
  background-color: var(--backgroundColor);
  color: var(--textColor);
}

.calendar-setting {
  display: flex;
  justify-content: right;
  align-items: center;
  margin-bottom: 1rem;
}
.calendar-controller {
  display: flex;
  justify-content: center;
  align-items: center;
  margin-bottom: 0.5rem;
}

.calendar-body {
  display: grid;
  grid-template-columns: repeat(7, 1fr);
  /* gap: 0.5rem; */
}

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

.calender-cell-header {
  font-weight: bold;
}

.calendar-cell-diluted {
  color: grey;
}
.calendar-cell-highlighted {
  border: 3px dashed var(--highlightSelected);
}
.calendar-cell-selected {
  border: 3px solid var(--highlightSelected);
}
/* calendar end */

/* tag area begin*/
.tag_area {
  display: flex;
  flex-direction: row;
  justify-content: space-between;
  align-items: flex-start;
  width: 100%;
  background-color: var(--backgroundColor);
  color: var(--textColor);
}

.multi_pick,
.edit {
  flex-shrink: 0;
}

.col_checkbox {
  display: flex;
  flex-direction: column; /* 将子元素垂直排列 */
  align-items: center; /* 子元素在水平方向居中对齐 */
}

li,
ul {
  list-style: none;
  margin: 0;
  padding: 0;
}

.tags_container {
  display: flex;
  flex-flow: row wrap;
  flex-grow: 1;
  justify-content: center;
  align-items: flex-start;
  align-content: flex-start;
  /* width: 100%; */
  height: 100%;
  max-height: 6rem;
  overflow-y: scroll;
}

.tag {
  padding: 2px 5px;
  margin: 10px;
  font-size: 18px;
  border: 3px solid var(--backgroundColor);
  cursor: pointer;
}

.custom_tag {
  position: relative;
}

.selected-tag {
  border: 3px solid var(--highlightSelected);
}

.delete-btn {
  position: absolute;
  top: -10px;
  left: -10px;
  background: red;
  color: var(--textColor);
  border: none;
  border-radius: 50%;
  cursor: pointer;
  width: 20px;
  height: 20px;
  display: flex;
  align-items: center;
  justify-content: center;
}

.modal {
  display: block;
  position: fixed;
  /* left: 0;
  top: 0;
  width: 100%; */
  width: 200px;
  top: 50%;
  left: 50%;
  background-color: rgba(0, 0, 0, 0.8);
  z-index: 1;
}

.modal-content {
  /* position:absolute; */
  /* background-color: var(--backgroundColor); */
  /* padding: 20px; */
  border-radius: 4px;
  color: var(--textColor);
  transform: translate(-50%, -50%) linear 1s;
}

.close-button {
  float: right;
  cursor: pointer;
}

.sepan {
  /* background-color: var(--backgroundColor); */
}

input[type="radio"] {
  display: none;
}

.color {
  display: inline-block;
  position: relative;
  box-sizing: border-box;
  width: 20px;
  height: 20px;
  border-radius: 50%;
  margin: 0 5px;
  cursor: pointer;
  text-align: center;
}

.color-selected {
  border: 3px solid var(--highlightSelected);
}
/* .color::after {
  content: "";
  display: none;
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  width: 12px; 
  height: 12px; 
  background-color: #3498db;
  border-radius: 50%;
} */

/* 当radio按钮被选中时，显示内圆 */
/* input[type="radio"]:checked + .color::after {
  display: block;
} */
/* tag area end*/

/* operation area begin */
.operation-area {
  display: flex;
  flex-flow: row wrap;
  justify-content: space-between;
  align-items: center;
  width: 100%;
  background-color: var(--backgroundColor);
  color: var(--textColor);
}
.btn-operation {
  width: 50%;
  padding: 5px;
  font-size: 18px;
  cursor: pointer;
}
/* operation area end */
</style>
