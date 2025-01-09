<template>
  <div class="game-container">
    <!-- 游戏区域 -->
    <div class="game-section">
      <div class="game-board" 
           :style="{ width: boardSize + 'px', height: boardSize + 'px' }"
           @keydown="handleKeyPress" 
           @touchstart="handleTouchStart"
           @touchmove="handleTouchMove"
           @touchend="handleTouchEnd"
           tabindex="0"
           ref="gameBoard">
        <!-- 蛇身 -->
        <div v-for="(segment, index) in snake" 
             :key="index" 
             class="snake-segment"
             :class="{
               'snake-head': index === 0,
               'snake-tail': index === snake.length - 1
             }"
             :style="{ 
               left: segment.x * gridSize + 'px', 
               top: segment.y * gridSize + 'px',
               transform: index === 0 ? `rotate(${getHeadRotation()}deg)` : 'none'
             }">
        </div>
        <!-- 食物 -->
        <div class="food"
             :style="{ left: food.x * gridSize + 'px', top: food.y * gridSize + 'px' }">
        </div>
      </div>
      <div class="current-score">
        得分: <span>{{ score }}</span>
      </div>
    </div>

    <!-- 控制区和排行榜并排 -->
    <div class="side-section">
      <div class="control-section">
        <div class="difficulty-selector">
          <span>选择难度</span>
          <div class="buttons-container">
            <button 
              v-for="level in difficulties" 
              :key="level.value"
              :class="['difficulty-btn', { active: currentDifficulty === level.value }]"
              :data-difficulty="level.value"
              @click="setDifficulty(level.value)"
              :disabled="isPlaying">
              {{ level.label }}
            </button>
          </div>
        </div>
        <button class="start-btn" @click="startGame" :disabled="isPlaying">
          {{ isPlaying ? '游戏中...' : '开始游戏' }}
        </button>
      </div>

      <div class="leaderboard">
        <h3>排行榜</h3>
        <div class="records-container">
          <div v-for="(score, index) in topScores" 
               :key="index" 
               class="record">
            <div class="rank">{{ index + 1 }}</div>
            <div class="score-value">{{ score }}分</div>
          </div>
          <div v-if="topScores.length === 0" class="record empty">
            暂无记录
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted } from 'vue';

const gridSize = 20;
const boardSize = 360;
const gridCount = 18;

const snake = ref([
  { x: 3, y: 0 }, 
  { x: 2, y: 0 }, 
  { x: 1, y: 0 }
]);
const food = ref({ x: 5, y: 5 });
const direction = ref('right');
const isPlaying = ref(false);
const score = ref(0);
const gameBoard = ref(null);
const topScores = ref([]);
let gameInterval;

// 难度设置
const difficulties = [
  { value: 'easy', label: '简单', speed: 200, scoreMultiplier: 1, color: '#4CAF50' },
  { value: 'medium', label: '中等', speed: 150, scoreMultiplier: 1.5, color: '#2196F3' },
  { value: 'hard', label: '困难', speed: 100, scoreMultiplier: 2, color: '#f44336' }
];
const currentDifficulty = ref('medium');

// 设置难度
const setDifficulty = (difficulty) => {
  currentDifficulty.value = difficulty;
};

// 获取当前难度配置
const getCurrentDifficultyConfig = () => {
  return difficulties.find(d => d.value === currentDifficulty.value);
};

// 获取蛇头旋转角度
const getHeadRotation = () => {
  switch (direction.value) {
    case 'up': return -90;
    case 'down': return 90;
    case 'left': return 180;
    case 'right': return 0;
  }
};

// 加载历史最高分
const loadTopScores = () => {
  topScores.value = JSON.parse(localStorage.getItem('snakeTopScores') || '[]');
};

// 保存新的分数
const saveScore = (newScore) => {
  // 获取当前排行榜
  const currentScores = JSON.parse(localStorage.getItem('snakeTopScores') || '[]');
  // 添加新分数
  currentScores.push(newScore);
  // 排序并只保留前三名
  const sortedScores = currentScores.sort((a, b) => b - a).slice(0, 3);
  // 保存到本地存储
  localStorage.setItem('snakeTopScores', JSON.stringify(sortedScores));
  // 更新显示
  topScores.value = sortedScores;
};

// 开始游戏
const startGame = () => {
  isPlaying.value = true;
  score.value = 0;
  direction.value = 'right';
  snake.value = [
    { x: Math.floor(gridCount / 2), y: Math.floor(gridCount / 2) },
    { x: Math.floor(gridCount / 2) - 1, y: Math.floor(gridCount / 2) },
    { x: Math.floor(gridCount / 2) - 2, y: Math.floor(gridCount / 2) }
  ];
  clearInterval(gameInterval);
  generateFood();
  gameBoard.value.focus();
  const difficultyConfig = getCurrentDifficultyConfig();
  gameInterval = setInterval(moveSnake, difficultyConfig.speed);
};

// 结束游戏
const endGame = () => {
  isPlaying.value = false;
  clearInterval(gameInterval);
  gameInterval = null;
  if (score.value > 0) { // 只保存大于0的分数
    saveScore(score.value);
  }
  alert(`游戏结束！得分：${score.value}`);
  loadTopScores(); // 重新加载排行榜
};

// 生成食物
const generateFood = () => {
  const newFood = {
    x: Math.floor(Math.random() * (gridCount - 2)) + 1,
    y: Math.floor(Math.random() * (gridCount - 2)) + 1
  };

  // 额外的安全检查
  if (newFood.x < 1 || newFood.x >= gridCount - 1 || newFood.y < 1 || newFood.y >= gridCount - 1) {
    generateFood();
    return;
  }

  // 确保食物不会生成在蛇身上
  if (snake.value.some(segment => segment.x === newFood.x && segment.y === newFood.y)) {
    generateFood();
    return;
  }
  food.value = newFood;
};

// 移动蛇
const moveSnake = () => {
  const head = { ...snake.value[0] };
  
  switch (direction.value) {
    case 'up': head.y--; break;
    case 'down': head.y++; break;
    case 'left': head.x--; break;
    case 'right': head.x++; break;
  }

  // 边界检测
  if (head.x >= gridCount || head.x < 0 || head.y >= gridCount || head.y < 0) {
    endGame();
    return;
  }

  // 检查是否会撞到自己（不包括尾巴，因为它会移动）
  if (snake.value.slice(0, -1).some(segment => segment.x === head.x && segment.y === head.y)) {
    endGame();
    return;
  }

  snake.value.unshift(head);

  // 检查是否吃到食物
  if (head.x === food.value.x && head.y === food.value.y) {
    const difficultyConfig = getCurrentDifficultyConfig();
    score.value += Math.round(10 * difficultyConfig.scoreMultiplier);
    generateFood();
  } else {
    snake.value.pop();
  }
};

// 处理按键
const handleKeyPress = (e) => {
  if (!isPlaying.value) return;

  switch (e.key) {
    case 'ArrowUp':
      if (direction.value !== 'down') direction.value = 'up';
      break;
    case 'ArrowDown':
      if (direction.value !== 'up') direction.value = 'down';
      break;
    case 'ArrowLeft':
      if (direction.value !== 'right') direction.value = 'left';
      break;
    case 'ArrowRight':
      if (direction.value !== 'left') direction.value = 'right';
      break;
  }
};

// 触摸控制相关变量
const touchStartX = ref(0);
const touchStartY = ref(0);
const minSwipeDistance = 20; // 降低滑动触发阈值

// 处理触摸开始
const handleTouchStart = (e) => {
  if (!isPlaying.value) return;
  e.preventDefault();
  touchStartX.value = e.touches[0].clientX;
  touchStartY.value = e.touches[0].clientY;
};

// 处理触摸移动
const handleTouchMove = (e) => {
  if (!isPlaying.value) return;
  e.preventDefault();
  
  const touchEndX = e.touches[0].clientX;
  const touchEndY = e.touches[0].clientY;
  
  const deltaX = touchEndX - touchStartX.value;
  const deltaY = touchEndY - touchStartY.value;
  
  // 判断滑动方向
  if (Math.abs(deltaX) > Math.abs(deltaY)) {
    // 水平滑动
    if (deltaX > 0 && direction.value !== 'left') {
      direction.value = 'right';
    } else if (deltaX < 0 && direction.value !== 'right') {
      direction.value = 'left';
    }
  } else {
    // 垂直滑动
    if (deltaY > 0 && direction.value !== 'up') {
      direction.value = 'down';
    } else if (deltaY < 0 && direction.value !== 'down') {
      direction.value = 'up';
    }
  }
};

const handleTouchEnd = (e) => {
  if (!isPlaying.value) return;
  e.preventDefault();
};

onMounted(() => {
  gameBoard.value.focus();
  loadTopScores();
});

onUnmounted(() => {
  clearInterval(gameInterval);
});
</script>

<style scoped>
.game-container {
  display: flex;
  flex-direction: row;
  justify-content: center;
  align-items: flex-start;
  gap: 20px;
  padding: 20px;
  max-width: 800px;
  margin: 0 auto;
}

.game-section {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 10px;
}

.side-section {
  display: flex;
  flex-direction: column;
  gap: 15px;
  width: 250px;
}

.control-section {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 12px;
  width: 100%;
}

.leaderboard {
  background: linear-gradient(145deg, #ffffff, #f5f5f5);
  border-radius: 12px;
  padding: 15px;
  box-shadow: 0 4px 8px rgba(0,0,0,0.1);
  width: 100%;
}

.game-board {
  position: relative;
  background-color: #eee;
  border: 3px solid #333;
  outline: none;
  border-radius: 8px;
  overflow: hidden;
  width: 360px !important;
  height: 360px !important;
  box-shadow: inset 0 0 10px rgba(0,0,0,0.1);
}

.snake-segment {
  position: absolute;
  width: 18px;
  height: 18px;
  background-color: #4CAF50;
  border: 2px solid #45a049;
  border-radius: 2px;
  box-shadow: 0 0 2px rgba(0,0,0,0.2);
}

.snake-head {
  background-color: #2E7D32;
  border-radius: 4px;
  z-index: 2;
}

/* 眼睛 */
.snake-head:before,
.snake-head:after {
  content: '';
  position: absolute;
  width: 4px;
  height: 4px;
  background-color: white;
  border-radius: 50%;
  top: 4px;
}

.snake-head:before {
  left: 4px;
}

.snake-head:after {
  right: 4px;
}

.snake-tail {
  background-color: #81C784;
  border-radius: 8px;
}

.food {
  position: absolute;
  width: 18px;
  height: 18px;
  background-color: #f44336;
  border: 2px solid #d32f2f;
  border-radius: 50%;
  box-shadow: 0 0 4px rgba(244,67,54,0.5);
}

.score {
  font-size: 24px;
  font-weight: bold;
}

.leaderboard h3 {
  color: #333;
  text-align: center;
  margin-bottom: 10px;
  font-size: 1.1em;
}

.records-container {
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.record {
  display: flex;
  align-items: center;
  padding: 6px 10px;
  background-color: white;
  border-radius: 6px;
  transition: transform 0.2s;
}

.record:hover {
  transform: translateX(5px);
}

.rank {
  font-size: 1em;
  font-weight: bold;
  color: #666;
  margin-right: 15px;
}

.score-value {
  font-size: 1em;
  color: #2196F3;
  font-weight: bold;
}

.current-score {
  font-size: 1.2em;
  margin-top: 5px;
}

.start-btn {
  width: 100%;
  padding: 8px;
  font-size: 0.95em;
  background: linear-gradient(145deg, #2196F3, #1976D2);
  color: white;
  border: none;
  border-radius: 6px;
  cursor: pointer;
  transition: all 0.3s ease;
  box-shadow: 0 2px 4px rgba(0,0,0,0.1);
}

.start-btn:hover:not(:disabled) {
  transform: translateY(-2px);
  box-shadow: 0 6px 12px rgba(0,0,0,0.15);
}

.start-btn:disabled {
  background: linear-gradient(145deg, #ccc, #999);
  cursor: not-allowed;
  transform: none;
}

.difficulty-selector {
  width: 100%;
  padding: 10px;
}

.difficulty-selector span {
  font-size: 14px;
  font-weight: bold;
  color: #333;
  margin-bottom: 8px;
  display: block;
}

.difficulty-selector .buttons-container {
  display: flex;
  gap: 10px;
  width: 100%;
  justify-content: center;
}

.difficulty-btn {
  flex: 1;
  max-width: 80px;
  padding: 6px 10px;
  border: 1px solid transparent;
  background-color: white;
  font-weight: bold;
  font-size: 0.85em;
  cursor: pointer;
  border-radius: 6px;
  transition: all 0.3s ease;
  position: relative;
  overflow: hidden;
}

.difficulty-btn:not(:disabled)::after {
  content: '';
  position: absolute;
  bottom: 0;
  left: 0;
  width: 100%;
  height: 3px;
  background-color: currentColor;
  transform: scaleX(0);
  transition: transform 0.3s ease;
}

.difficulty-btn:hover:not(:disabled)::after {
  transform: scaleX(1);
}

.difficulty-btn.active {
  transform: translateY(-2px);
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
}

.difficulty-btn:disabled {
  opacity: 0.7;
  cursor: not-allowed;
  transform: none;
}

/* 难度按钮颜色 */
.difficulty-btn[data-difficulty="easy"] {
  color: #4CAF50;
}

.difficulty-btn[data-difficulty="medium"] {
  color: #2196F3;
}

.difficulty-btn[data-difficulty="hard"] {
  color: #f44336;
}

.difficulty-btn.active[data-difficulty="easy"] {
  background-color: #E8F5E9;
}

.difficulty-btn.active[data-difficulty="medium"] {
  background-color: #E3F2FD;
}

.difficulty-btn.active[data-difficulty="hard"] {
  background-color: #FFEBEE;
}

/* 响应式布局 */
@media (max-width: 768px) {
  .game-container {
    flex-direction: column;
    align-items: center;
    padding: 10px;
    gap: 15px;
  }

  .game-board {
    touch-action: none;
    border: 2px solid #333;
    margin: 0 auto;
  }

  .side-section {
    width: 100%;
    max-width: 360px;
    gap: 10px;
  }

  .difficulty-selector {
    background: #fff;
    border-radius: 8px;
    padding: 10px;
    box-shadow: 0 2px 4px rgba(0,0,0,0.1);
  }

  .difficulty-btn {
    padding: 10px;
    margin: 0 5px;
    min-width: 70px;
    border-radius: 20px;
    font-size: 14px;
  }

  .start-btn {
    width: 90%;
    padding: 12px;
    font-size: 1.1em;
    border-radius: 25px;
    margin: 10px auto;
  }

  .current-score {
    font-size: 1.5em;
    margin: 10px 0;
    font-weight: bold;
  }

  .leaderboard {
    margin-top: 10px;
    padding: 15px;
    border-radius: 15px;
  }

  .record {
    padding: 10px 15px;
    margin: 5px 0;
    border-radius: 10px;
  }
}

/* 防止移动端选中文本 */
* {
  -webkit-tap-highlight-color: transparent;
  -webkit-touch-callout: none;
  user-select: none;
}
</style> 