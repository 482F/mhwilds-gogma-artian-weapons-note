<template>
  <div class="app">
    <div class="header">mhwilds 巨戟アーティアメモ</div>
    <tabs
      class="tabs"
      :tabs="[
        { key: 'bonus-note', label: '巨戟復元強化' },
        { key: 'skill-note', label: 'スキルの再付与' },
        { key: 'setting', label: '設定' },
      ]"
    >
      <template v-slot:bonus-note>
        <note
          v-model:columns="bonusColumns"
          v-model:stock="stock"
          mode="bonus"
        />
      </template>
      <template v-slot:skill-note>
        <note
          v-model:columns="skillColumns"
          v-model:stock="stock"
          mode="skill"
        />
      </template>
      <template v-slot:setting>
        <setting />
      </template>
    </tabs>
  </div>
</template>

<script setup lang="ts">
import { useLocalStorage } from './composable/local-storage-usable'

import Note, { type Column, type Stock } from './feature/note/component.vue'
import Setting from './feature/setting/component.vue'

import Tabs from './component/tabs/component.vue'

const bonusColumns = useLocalStorage<Column[]>('bonus-columns', [])
const skillColumns = useLocalStorage<Column[]>('skill-columns', [])
const stock = useLocalStorage<Stock>('stock', {
  oricalcite: 0,
  tarredDevice: {},
})
</script>

<style scoped>
.app {
  font-size: 16px;
  padding: 2% 5%;
  width: 100%;
  height: 100%;
  overflow: auto;

  background-color: white;

  display: grid;
  grid-template-areas:
    'header'
    'content';
  grid-template-rows: max-content 1fr;
  gap: 32px;

  > .header {
    grid-area: header;
    font-size: 32px;
  }
  > .content {
    grid-area: content;
  }
}
</style>

<style>
html {
  --c-active: #6c8cff;
  --c-active-bg: #eef7ff;
  --c-hover-bg: #fafafa;
  --c-border: #e3e3e3;
}
body {
  margin: 0;
}
* {
  font-family: 'Cica';
  box-sizing: border-box;
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
