<template>
  <div class="tabs">
    <div class="tab-bar">
      <btn
        v-for="{ key, label } of tabs"
        :key="key"
        @click="currentKey = key"
        :class="{
          tab: true,
          current: currentKey === key,
        }"
      >
        {{ label }}
      </btn>
    </div>
    <div class="content">
      <slot :name="tabs.find(({ key }) => key === currentKey)?.key" />
    </div>
  </div>
</template>

<script lang="ts" setup>
import Btn from '../button/component.vue'

const props = defineProps<{
  tabs: readonly { key: symbol | number | string; label: string }[]
}>()
defineSlots<{ [key in string]: () => any }>()
const currentKey = defineModel<symbol | number | string | undefined>(
  'currentKey',
  {
    default: undefined,
  }
)
currentKey.value ??= props.tabs[0]?.key
</script>

<style scoped>
.tabs {
  display: grid;
  grid-template-areas:
    'tab-bar'
    'content';
  grid-template-rows: max-content 1fr;

  > .tab-bar {
    grid-area: tab-bar;
    padding: 0 0.5px;
    > .tab {
      border: solid 1px var(--c-border);
      background-color: white;
      &:hover {
        background-color: var(--c-hover-bg);
      }
      &.current {
        color: var(--c-active);
      }

      margin: 0 -0.5px;
      font-size: 1.1rem;
      padding: 0.8rem 1.5rem;
    }
  }
  > .content {
    grid-area: content;
  }
}
</style>
