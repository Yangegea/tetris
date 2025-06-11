<template>
  <div class="tetris-container">
    <div class="game-info">
      <div class="score">积分: {{ score }}</div>
      <div class="level">等级: {{ level }}</div>
      <div class="next-piece">
        <h3>下一个方块</h3>
        <div class="next-piece-board">
          <div v-for="(row, y) in nextPieceBoard" :key="y" class="row">
            <div v-for="(cell, x) in row" 
                 :key="x" 
                 class="cell"
                 :class="{ filled: cell }">
            </div>
          </div>
        </div>
      </div>
    </div>
    
    <div class="game-board">
      <div v-for="(row, y) in displayBoard" :key="y" class="row">
        <div v-for="(cell, x) in row" 
             :key="x" 
             class="cell"
             :class="{ 
               'filled': cell === 1,
               'current': cell === 2 
             }">
        </div>
      </div>
    </div>

    <div class="leaderboard">
      <h3>排行榜</h3>
      <ol>
        <li v-for="(score, index) in topScores" :key="index">
          {{ score }}积分
        </li>
      </ol>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted, computed } from 'vue'

// 游戏配置
const BOARD_WIDTH = 10
const BOARD_HEIGHT = 20
const INITIAL_SPEED = 1000

// 游戏状态
const board = ref(Array(BOARD_HEIGHT).fill().map(() => Array(BOARD_WIDTH).fill(0)))
const score = ref(0)
const level = ref(1)
const topScores = ref([])
const nextPieceBoard = ref(Array(4).fill().map(() => Array(4).fill(0)))

// 方块形状定义
const TETROMINOES = {
  I: [[1, 1, 1, 1]],
  O: [[1, 1], [1, 1]],
  T: [[0, 1, 0], [1, 1, 1]],
  L: [[1, 0], [1, 0], [1, 1]],
  J: [[0, 1], [0, 1], [1, 1]],
  S: [[0, 1, 1], [1, 1, 0]],
  Z: [[1, 1, 0], [0, 1, 1]]
}

// 当前方块状态
const currentPiece = ref(null)
const currentPosition = ref({ x: 0, y: 0 })
const nextPiece = ref(null)

// 添加计算属性来合并当前方块和游戏板
const displayBoard = computed(() => {
  const display = board.value.map(row => [...row])
  
  if (currentPiece.value) {
    currentPiece.value.forEach((row, y) => {
      row.forEach((cell, x) => {
        if (cell) {
          const boardY = currentPosition.value.y + y
          const boardX = currentPosition.value.x + x
          if (boardY >= 0 && boardY < BOARD_HEIGHT && boardX >= 0 && boardX < BOARD_WIDTH) {
            display[boardY][boardX] = 2
          }
        }
      })
    })
  }
  
  return display
})

// 初始化游戏
function initGame() {
  score.value = 0
  level.value = 1
  board.value = Array(BOARD_HEIGHT).fill().map(() => Array(BOARD_WIDTH).fill(0))
  generateNewPiece()
  startGame()
}

// 生成新方块
function generateNewPiece() {
  const pieces = Object.keys(TETROMINOES)
  if (!nextPiece.value) {
    nextPiece.value = TETROMINOES[pieces[Math.floor(Math.random() * pieces.length)]]
  }
  currentPiece.value = nextPiece.value
  nextPiece.value = TETROMINOES[pieces[Math.floor(Math.random() * pieces.length)]]
  currentPosition.value = {
    x: Math.floor((BOARD_WIDTH - currentPiece.value[0].length) / 2),
    y: 0
  }
  updateNextPieceBoard()
}

// 更新下一个方块预览
function updateNextPieceBoard() {
  nextPieceBoard.value = Array(4).fill().map(() => Array(4).fill(0))
  nextPiece.value.forEach((row, y) => {
    row.forEach((cell, x) => {
      if (cell) {
        nextPieceBoard.value[y + 1][x + 1] = cell
      }
    })
  })
}

// 检查碰撞
function checkCollision(piece, position) {
  return piece.some((row, y) => {
    return row.some((cell, x) => {
      if (!cell) return false
      const newX = position.x + x
      const newY = position.y + y
      return newX < 0 || newX >= BOARD_WIDTH || 
             newY >= BOARD_HEIGHT ||
             (newY >= 0 && board.value[newY][newX])
    })
  })
}

// 合并方块到板面
function mergePiece() {
  currentPiece.value.forEach((row, y) => {
    row.forEach((cell, x) => {
      if (cell) {
        const boardY = currentPosition.value.y + y
        const boardX = currentPosition.value.x + x
        if (boardY >= 0) {
          board.value[boardY][boardX] = 1
        }
      }
    })
  })
  checkLines()
  generateNewPiece()
  if (checkCollision(currentPiece.value, currentPosition.value)) {
    gameOver()
  }
}

// 检查并清除完整行
function checkLines() {
  let linesCleared = 0
  board.value.forEach((row, y) => {
    if (row.every(cell => cell)) {
      board.value.splice(y, 1)
      board.value.unshift(Array(BOARD_WIDTH).fill(0))
      linesCleared++
    }
  })
  if (linesCleared > 0) {
    updateScore(linesCleared)
  }
}

// 更新分数
function updateScore(linesCleared) {
  const points = {
    1: 100,
    2: 300,
    3: 500,
    4: 800
  }
  score.value += points[linesCleared] * level.value
  level.value = Math.floor(score.value / 1000) + 1
}

// 游戏结束处理
function gameOver() {
  updateLeaderboard()
  alert(`游戏结束！积分：${score.value}`)
  initGame()
}

// 更新排行榜
function updateLeaderboard() {
  topScores.value.push(score.value)
  topScores.value.sort((a, b) => b - a)
  topScores.value = topScores.value.slice(0, 5)
  localStorage.setItem('tetrisTopScores', JSON.stringify(topScores.value))
}

// 键盘控制
function handleKeydown(event) {
  switch(event.key) {
    case 'ArrowLeft':
      movePiece(-1, 0)
      break
    case 'ArrowRight':
      movePiece(1, 0)
      break
    case 'ArrowDown':
      movePiece(0, 1)
      break
    case 'ArrowUp':
      rotatePiece()
      break
  }
}

// 移动方块
function movePiece(dx, dy) {
  const newPosition = {
    x: currentPosition.value.x + dx,
    y: currentPosition.value.y + dy
  }
  if (!checkCollision(currentPiece.value, newPosition)) {
    currentPosition.value = newPosition
  } else if (dy > 0) {
    mergePiece()
  }
}

// 旋转方块
function rotatePiece() {
  const rotated = currentPiece.value[0].map((_, i) =>
    currentPiece.value.map(row => row[i]).reverse()
  )
  if (!checkCollision(rotated, currentPosition.value)) {
    currentPiece.value = rotated
  }
}

// 游戏循环
let gameInterval
function startGame() {
  if (gameInterval) clearInterval(gameInterval)
  gameInterval = setInterval(() => {
    movePiece(0, 1)
  }, INITIAL_SPEED / level.value)
}

// 生命周期钩子
onMounted(() => {
  topScores.value = JSON.parse(localStorage.getItem('tetrisTopScores') || '[]')
  window.addEventListener('keydown', handleKeydown)
  initGame()
})

onUnmounted(() => {
  window.removeEventListener('keydown', handleKeydown)
  if (gameInterval) clearInterval(gameInterval)
})
</script>

<style scoped>
.tetris-container {
  display: flex;
  justify-content: center;
  gap: 20px;
  padding: 20px;
}

.game-board {
  border: 2px solid #333;
  background: #f0f0f0;
}

.row {
  display: flex;
}

.cell {
  width: 30px;
  height: 30px;
  border: 1px solid #ccc;
}

.cell.filled {
  background: #42b883;
}

.cell.current {
  background: #3498db;
}

.game-info {
  padding: 20px;
  background: #f8f8f8;
  border-radius: 8px;
}

.next-piece-board {
  width: 120px;
  height: 120px;
  border: 1px solid #ccc;
  margin-top: 10px;
}

.leaderboard {
  padding: 20px;
  background: #f8f8f8;
  border-radius: 8px;
}

.score, .level {
  font-size: 1.2em;
  margin-bottom: 10px;
}
</style> 