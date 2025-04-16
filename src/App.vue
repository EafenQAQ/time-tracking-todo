<template>
  <div class="container">
    <h1>时间追踪 Todo List</h1>
    <TodoList />
  </div>
</template>

<script setup>
import TodoList from "./components/TodoList.vue";
</script>

<style>
.container {
  max-width: 800px;
  margin: 0 auto;
  padding: 20px;
  font-family: Arial, sans-serif;
}

h1 {
  text-align: center;
  color: #333;
  margin-bottom: 30px;
}
</style>