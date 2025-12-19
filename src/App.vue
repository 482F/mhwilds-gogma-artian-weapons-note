<template>
  <div class="app">
    <div class="header">mhwilds 巨戟アーティアメモ</div>
    <tabs class="tabs" v-model:current-key="currentTab" :tabs="tabDefs">
      <template v-slot:bonus-note>
        <note
          v-model:columns="bonus.columns"
          v-model:stock="stock"
          v-model:processed-num="bonus.processedNum"
          v-model:favorite-skills="favoriteSkills"
          mode="bonus"
        />
      </template>
      <template v-slot:skill-note>
        <note
          v-model:columns="skill.columns"
          v-model:stock="stock"
          v-model:processed-num="skill.processedNum"
          v-model:favorite-skills="favoriteSkills"
          mode="skill"
        />
      </template>
    </tabs>
  </div>
</template>

<script setup lang="ts">
import { useLocalStorage } from './composable/local-storage-usable'

import Note, {
  type Column,
  type Stock,
  type FavoriteSkills,
} from './feature/note/component.vue'

import Tabs from './component/tabs/component.vue'

const tabDefs = [
  { key: 'bonus-note', label: '巨戟復元強化' },
  { key: 'skill-note', label: 'スキルの再付与' },
] as const
const currentTab = useLocalStorage<(typeof tabDefs)[number]['key']>(
  'current-tab',
  'bonus-note'
)
const bonus = useLocalStorage<{
  columns: Column[]
  processedNum: number
}>('bonus', { columns: [], processedNum: 0 })
const skill = useLocalStorage<{
  columns: Column[]
  processedNum: number
}>('skill', {
  columns: [],
  processedNum: 0,
})
const stock = useLocalStorage<Stock>('stock', {
  oricalcite: 0,
  tarredDevice: {},
})
const favoriteSkills = useLocalStorage<FavoriteSkills>('favorite-skills', {
  series: [],
  group: [],
})
</script>

<style scoped>
.app {
  font-size: 16px;
  padding: 2% 5%;
  width: 100%;
  height: 100%;

  background-color: white;

  display: grid;
  grid-template-areas:
    'header'
    'content';
  grid-template-rows: max-content 1fr;
  > * {
    min-width: 0;
  }
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
