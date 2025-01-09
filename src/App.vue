<template>
  <div class="app">
    <SnakeGame />
  </div>
</template>

<script setup>
import SnakeGame from './components/SnakeGame.vue'
</script>

<style>
.app {
  display: flex;
  justify-content: center;
  align-items: center;
  min-height: 100vh;
  margin: 0;
  background-color: #f0f2f5;
}
</style> 