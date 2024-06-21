<template>
  <div>
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
            x
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
          <button @click="addTag(newTag.name)" v-if="isAdding">Add</button>
          <button @click="closeAddTagModal">Cancel</button>
        </div>
      </div>
    </div>
    <button>Mark</button>
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
// 用户tags
const tags = reactive([
  {
    name: "看看你的",
    color: COLORS[0].rgb,
    isSelected: false,
  },
  //   {
  //     name: "看看你的",
  //     color: COLORS[0].rgb,
  //     order: 1,
  //   },
  //   {
  //     name: "看看你的",
  //     color: COLORS[0].rgb,
  //     order: 2,
  //   },
]);
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
  for (
    let i = daysInLastMonth - last_month_rest_count + 1;
    i <= daysInLastMonth;
    i++
  ) {
    datesArray.push({
      date: new Date(year, month - 1, i),
      isToday: false,
      isCurrentMonth: false,
      // TODO: complete record of last_month_rest
      activities: [],
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
      // TODO: complete record of last_month_rest
      activities: [],
    });
  }

  // 插入下月日期
  const daysArray_len = datesArray.length;
  for (let i = 1; i <= 42 - daysArray_len; i++) {
    datesArray.push({
      date: new Date(year, month + 1, i),
      isToday: false,
      isCurrentMonth: false,
      // TODO: complete record of last_month_rest
      activities: [],
    });
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
const addTag = (name, color = COLORS[0].rgb) => {
  if (name !== "") {
    newTag = {
      name: name,
      color: color,
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
</script>

<style scoped>
/* calendar begin */
.calendar {
  width: 500px;
  margin: 0;
  background-color: black;
  color: #f0f0f0;
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
  border: 3px dashed whitesmoke;
}
.calendar-cell-selected {
  border: 3px solid whitesmoke;
}
/* calendar end */

/* tag area begin*/
.tag_area {
  display: flex;
  flex-direction: row;
  justify-content: space-between;
  align-items: flex-start;
  width: 500px;
  background-color: black;
  color: white;
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
  border: 3px solid black;
}

.custom_tag {
  position: relative;
}

.selected-tag {
  border: 3px solid whitesmoke;
}

.delete-btn {
  position: absolute;
  top: -10px;
  left: -10px;
  background: red;
  color: white;
  border: none;
  /* border-radius: 50%; */
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
  left: 0;
  top: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.5);
  z-index: 1;
}

.modal-content {
  position: absolute;
  background-color: #fff;
  padding: 20px;
  border-radius: 4px;
  width: 300px;
  top: 50%;
  left: 50%;
  color: black;
  transform: translate(-50%, -50%);
}

.close-button {
  float: right;
  cursor: pointer;
}

.sepan {
  background-color: black;
}
.color {
  width: 50px;
  height: 50px;
  border-radius: 50%;
  margin: 10px;
  display: inline-block;
  color: rgb(255, 255, 255);
  text-align: center;
  /* color: #d3d301; */
}
/* tag area end*/
</style>
