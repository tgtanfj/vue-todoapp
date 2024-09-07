<template>
  <div id="app">
    <div
      class="absolute inset-0 -z-10 h-full w-full bg-[#4ec5c1] bg-[radial-gradient(#e5e7eb_1px,transparent_1px)] [background-size:16px_16px]"
    >
      <TaskTodo></TaskTodo>
    </div>
  </div>
</template>

<script setup>
import TaskTodo from './components/TaskTodo.vue'
</script>
