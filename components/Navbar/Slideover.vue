<script setup lang="ts">
//Props
defineProps<{
  open: boolean
}>()

//Emits
const emit = defineEmits<{
  (e: 'close'): void
}>()

//Functions
const moveToPage = (page: string): void => {
  useRouter().push(page)
  emit('close')
}
</script>

<template>
  <USlideover
    side="left"
    title="Navigation Menu"
    aria-labelledby="dialog-title"
    aria-describedby="dialog-description"
    description="Use this menu to navigate through the pages."
    :close="false"
    :open="open"
    @update:open="
      (value) => {
        if (!value) emit('close')
      }
    "
  >
    <template #header>
      <div class="flex w-full items-center justify-between">
        <div class="text-2xl">Navigation</div>
        <UButton
          icon="mdi:arrow-left"
          variant="link"
          color="neutral"
          class="cursor-pointer text-2xl"
          @click.stop="emit('close')"
        />
      </div>
    </template>
    <template #body>
      <div class="flex flex-col">
        <UButton
          label="Home"
          variant="link"
          class="cursor-pointer text-4xl font-bold text-blue-600 hover:text-gray-900"
          @click.stop="moveToPage('/')"
        />
        <UButton
          label="Bulletin"
          variant="link"
          class="cursor-pointer text-4xl font-bold text-blue-600 hover:text-gray-900"
          @click.stop="moveToPage('/bulletin')"
        />
        <UButton
          label="Events"
          variant="link"
          class="cursor-pointer text-4xl font-bold text-blue-600 hover:text-gray-900"
          @click.stop="moveToPage('/events')"
        />
        <UButton
          label="Executive Board"
          variant="link"
          class="cursor-pointer text-4xl font-bold text-blue-600 hover:text-gray-900"
          @click.stop="moveToPage('/executive-board')"
        />
        <UButton
          label="Members"
          variant="link"
          class="cursor-pointer text-4xl font-bold text-blue-600 hover:text-gray-900"
          @click.stop="moveToPage('/members')"
        />
        <UButton
          label="Resources"
          variant="link"
          class="cursor-pointer text-4xl font-bold text-blue-600 hover:text-gray-900"
          @click.stop="moveToPage('/resources')"
        />
        <UButton
          label="History"
          variant="link"
          class="cursor-pointer text-4xl font-bold text-blue-600 hover:text-gray-900"
          @click.stop="moveToPage('/history')"
        />
        <UButton
          label="Bylaws"
          variant="link"
          class="cursor-pointer text-4xl font-bold text-blue-600 hover:text-gray-900"
          @click.stop="moveToPage('/bylaws')"
        />
      </div>
    </template>
  </USlideover>
</template>
