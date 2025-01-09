<template>
  <div class="app">
    <div class="game-container">
      <h1 class="game-title">老K 贪吃蛇</h1>
      <SnakeGame />
    </div>
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

.game-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 20px;
}

.game-title {
  color: #2196F3;
  font-size: 32px;
  font-weight: bold;
  text-align: center;
  margin: 0;
  padding: 10px 30px;
  background: linear-gradient(145deg, #ffffff, #f5f5f5);
  border-radius: 12px;
  box-shadow: 0 4px 8px rgba(0,0,0,0.1);
  text-shadow: 1px 1px 2px rgba(0,0,0,0.1);
  letter-spacing: 2px;
  position: relative;
  overflow: hidden;
}

.game-title:hover {
  transform: translateY(-2px);
  transition: transform 0.3s ease;
}
</style> 
