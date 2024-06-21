<template>
  <div class="tag_area">
    <!-- 多选按钮 -->
    <!-- <div class="multi_pick">
      <label class="col_checkbox">
        <input type="checkbox" v-model="isMulti" />
        多选
      </label>
    </div> -->
    <!-- tags -->
    <!-- <div class="tags"> -->
    <ul class="tags_container">
      <li
        v-for="(tag, index) in tags"
        :key="index"
        class="tag custom_tag"
        :class="{ 'selected-tag': tag === selectedTag }"
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
          ×
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
  <!-- 色盘 -->
  <div class="sepan">
    <div
      v-for="color in colors"
      :key="color.name"
      :style="{ backgroundColor: color.rgb }"
      class="color"
    >
      {{ color.name }}
    </div>
  </div>
</template>

<script setup>
import { ref, reactive, computed } from "vue";

const colors = [
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
const isMulti = ref(Boolean(false));
const isEditing = ref(Boolean(false));
const isAdding = ref(Boolean(false));
const showModal = ref(Boolean(false));
const draggedTagIndex = ref(-1);
// let selectedTag = reactive({});
const selectedTag = ref({});
let newTag = reactive({
  name: "new tag",
  color: colors[0].rgb,
});
const tags = reactive([
  {
    name: "看看你的",
    color: colors[0].rgb,
    order: 0,
  },
  {
    name: "看看你的",
    color: colors[0].rgb,
    order: 1,
  },
  {
    name: "看看你的",
    color: colors[0].rgb,
    order: 2,
  },
  {
    name: "看看你的",
    color: colors[0].rgb,
    order: 3,
  },
  {
    name: "买水",
    color: colors[0].rgb,
    order: 4,
  },
  {
    name: "剪指甲",
    color: colors[0].rgb,
    order: 5,
  },
  {
    name: "剪头发",
    color: colors[0].rgb,
    order: 6,
  },
  {
    name: "跑步",
    color: colors[0].rgb,
    order: 7,
  },
  {
    name: "吃早饭",
    color: colors[0].rgb,
    order: 8,
  },
  {
    name: "洗澡",
    color: colors[0].rgb,
    order: 9,
  },
]);
const showAddTagModal = () => {
  isAdding.value = true;
  showModal.value = true;
};
const closeAddTagModal = () => {
  showModal.value = false;
  isAdding.value = false;
};
const addTag = (name, color = colors[0].rgb) => {
  if (name !== "") {
    newTag = {
      name: name,
      color: color,
    };
    tags.push(newTag);
    closeAddTagModal();
  }
};

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

const selectTag = (tag) => {
  // selectedTag = tag;
  selectedTag.value = tag;
};

const deleteTag = (index) => {
  tags.splice(index, 1);
};
</script>

<style scoped>
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
</style>
