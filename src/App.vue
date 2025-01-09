<template>
  <div class="app">
    <h1>老K 贪吃蛇</h1>
    <SnakeGame />
  </div>
</template>

<script setup>
import SnakeGame from './components/SnakeGame.vue';
</script>

<style>
.app {
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 20px;
}

h1 {
  color: #333;
  margin-bottom: 20px;
}
</style> 