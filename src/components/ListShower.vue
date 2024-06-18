<script setup>
import { ref, reactive, computed } from "vue";
// defineProps(['my_title','size']);
defineProps({
  my_title: {
    type: String,
    required: true,
  },
  size: {
    default: 12,
  },
});
const data = ref([
  {
    name: "Alice",
    age: 20,
    gender: "male",
  },
]);

const author = reactive({
  name: "John Doe",
  books: [
    "Vue 2 - Advanced Guide",
    "Vue 3 - Basic Guide",
    "Vue 4 - The Mystery",
  ],
});

// a computed ref
const publishedBooksMessage = computed(() => {
  console.log("computed");
  return author.books.length > 0 ? "Yes" : "No";
});
</script>

<template>
  <h1>{{ my_title }}&{{ size }}</h1>
  <div>
    <p v-for="(value, key, index) in data[0]">
      {{ value }},{{ key }},{{ index }}
    </p>
  </div>
  <p>Has published books? {{ publishedBooksMessage }}</p>
  <p>Has published books? {{ publishedBooksMessage }}</p>
</template>

<style lang="scss" scoped></style>
