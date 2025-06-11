<template>
  <div class="app">
    <h1>俄罗斯方块</h1>
    <Tetris />
  </div>
</template>

<script setup>
import Tetris from './components/Tetris.vue'
</script>

<style>
.app {
  text-align: center;
  font-family: Arial, sans-serif;
}

h1 {
  color: #42b883;
}
</style>
