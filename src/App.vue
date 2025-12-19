<template>
  <div class="app">
  </div>
</template>

<script setup lang="ts">
import Note from './feature/note/component.vue'
import Setting from './feature/setting/component.vue'
</script>

<style>
body {
  margin: 0;
}
* {
  font-family: 'Cica';
  &::-webkit-scrollbar-track {
    background-color: #eee;
  }
  &::-webkit-scrollbar {
    width: 10px;
    height: 10px;
  }
  &::-webkit-scrollbar-thumb {
    background-color: #777;
    border-radius: 30px;
  }
}
</style>
