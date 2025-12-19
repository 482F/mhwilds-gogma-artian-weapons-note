<template>
  <select class="select" v-model="modelValue">
    <option :value="undefined"></option>
    <hr />
    <template
      v-for="filteredItems of [
        items.filter((item) => favItemSet?.has(item)),
        items.filter((item) => !favItemSet?.has(item)),
      ]"
    >
      <option v-for="value of filteredItems" :value :key="value">
        {{ value }}
      </option>
      <hr />
    </template>
  </select>
</template>

<script lang="ts" setup generic="T extends string">
defineProps<{
  items: readonly T[]
  favItemSet?: Set<T>
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
