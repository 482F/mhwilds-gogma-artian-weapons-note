<template>
  <button class="button">
    <slot />
  </button>
</template>

<script lang="ts" setup></script>

<style scoped>
.button {
  border: solid 1px var(--c-border);
  background-color: white;
  &:hover {
    background-color: var(--c-hover-bg);
  }
}
</style>
