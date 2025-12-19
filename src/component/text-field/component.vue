<template>
  <input v-model="value" class="text-field" type="text" :placeholder />
</template>

<script lang="ts" setup>
defineProps<{
  placeholder?: string
}>()
const value = defineModel<string>()
</script>

<style scoped>
.text-field {
  font-size: 1rem;
  border: solid 1px var(--c-border);
  border-radius: 2px;
}
</style>
