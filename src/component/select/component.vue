<template>
  <select class="select" v-model="modelValue">
    <option :value="undefined"></option>
    <hr />
    <option v-for="value of items" :value :key="value">{{ value }}</option>
  </select>
</template>

<script lang="ts" setup generic="T extends string">
defineProps<{
  items: readonly T[]
}>()
const modelValue = defineModel<T | undefined>({
  default: undefined,
})
</script>

<style scoped>
.select {
  font-size: inherit;
  appearance: base-select;
  border: solid 1px var(--c-border);
  border-radius: 2px;
  > hr {
    margin: 0;
  }
  &::picker-icon {
    display: none;
  }
  justify-content: center;
  align-items: center;
  &:hover {
    background-color: var(--c-hover-bg);
  }
  > option {
    &::checkmark {
      display: none;
    }
    &:hover {
      background-color: var(--c-hover-bg);
    }
  }
}
::picker(select) {
  font-size: inherit;
  appearance: base-select;

  border-color: var(--c-border);
  border-radius: 2px;
}
</style>
