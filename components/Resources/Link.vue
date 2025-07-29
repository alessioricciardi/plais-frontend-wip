<script setup lang="ts">
//Props
defineProps<{
  label: string
  link: string
}>()
</script>

<template>
  <a
    :href="link"
    target="_blank"
    class="flex cursor-pointer items-start rounded-lg p-1 hover:bg-gray-100"
  >
    <UIcon name="mdi:link-variant" class="mr-1 flex-none text-lg" />
    <div class="text-sm">
      {{ label }}
    </div>
  </a>
</template>
