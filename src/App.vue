<script setup lang="ts">
/**
 * Ref used as state handler
 * onMounted to first render the LS items
 * Computed to create an ordered task array
 * Watch to observe LS changes
 */
import { ref, onMounted, computed, watch } from 'vue';

interface todo {
  content: string;
  category: string;
  createdAt: number;
  done: boolean;
}

const todos = ref<todo[]>([]);

const name = ref<string>('');

const inputContent = ref<string>('');
const inputCategory = ref<string | null>(null);

//Computing the todos in ascending order based on time created
const todoAsc = computed(() =>
  todos.value.sort((a, b) => b.createdAt - a.createdAt)
);

const addTodo = () => {
  if (inputContent.value.trim() === '' || inputCategory.value === null) return;
  todos.value.push({
    content: inputContent.value,
    category: inputCategory.value,
    createdAt: new Date().getTime(),
    done: false,
  });
  inputContent.value = '';
  inputCategory.value = null;
};

const removeTodo = (todo: todo) => {
  todos.value = todos.value.filter(v => v !== todo);
};

//Watching for a change in the input name, if it changes update LS
watch(name, newValue => {
  localStorage.setItem('name', newValue);
});

//Watching changes when a new todo is inserted
watch(
  todos,
  newVal => {
    localStorage.setItem('todos', JSON.stringify(newVal));
  },
  //Check for all changes, to recall
  { deep: true }
);

//When the component first mounts we retrieve the name item else we put an empty starting value
//Retrieve saved tasks as well
onMounted(() => {
  name.value = localStorage.getItem('name') || '';
  // @ts-ignore
  todos.value = JSON.parse(localStorage.getItem('todos')) || [];
});
</script>

<template>
  <main class="app">
    <section class="greeting">
      <h2 class="title">
        <!-- name correspons to ref property name -->
        What's up, <input type="text" placeholder="Name here" v-model="name" />
      </h2>
    </section>
    <section class="create-todo">
      <h3>CREATE TODO</h3>
      <form @submit.prevent="addTodo">
        <h4>What's on your todo list?</h4>
        <input
          type="text"
          placeholder="e.g. make a video"
          v-model="inputContent"
        />
        <h4>Pick a category</h4>
        <div class="options">
          <label>
            <input
              type="radio"
              name="category"
              id="category1"
              value="business"
              v-model="inputCategory"
            />
            <span class="bubble business"></span>
            <div>Business</div>
          </label>
          <label>
            <input
              type="radio"
              name="category"
              id="category2"
              value="personal"
              v-model="inputCategory"
            />
            <span class="bubble personal"></span>
            <div>Personal</div>
          </label>
        </div>
        <input type="submit" value="Add todo" />
      </form>
    </section>
    <section class="todo-list">
      <h3>Todo List</h3>
      <div class="list">
        <div
          v-for="todo in todoAsc"
          :class="`todo-item ${todo.done && 'done'}`"
        >
          <label>
            <input type="checkbox" v-model="todo.done" />
            <span :class="`bubble ${todo.category}`"></span>
          </label>
          <div class="todo-content">
            <input type="text" v-model="todo.content" />
          </div>
          <div class="actions">
            <button class="delete" @click="removeTodo(todo)">Delete</button>
          </div>
        </div>
      </div>
    </section>
  </main>
</template>

<style scoped></style>
