<script setup lang="ts">
// Props
const props = defineProps<{
  id: number
  name: string
  link: string
}>()

// Emits
const emit = defineEmits<{
  (e: 'edit', id: number): void
  (e: 'delete', id: number): void
}>()

// State
const loggedIn = useState<boolean>('loggedIn')
const showModalDelete = ref(false)

const toggleModalDelete = () => {
  showModalDelete.value = !showModalDelete.value
}

const confirmDelete = () => {
  emit('delete', props.id)
  toggleModalDelete()
}
</script>

<template>
  <UCard class="w-full">
    <div class="flex items-center justify-between">
      <div class="text-base font-medium text-gray-800">
        {{ props.name }}
      </div>

      <div class="flex items-center gap-2">
        <a
          :href="props.link"
          target="_blank"
          rel="noopener noreferrer"
          class="text-sm text-blue-600 underline hover:text-blue-800"
        >
          Visit
        </a>

        <!-- Edit icon -->
        <UButton
          v-if="loggedIn"
          class="cursor-pointer bg-amber-300 text-lg text-gray-900 hover:bg-amber-400"
          icon="mdi:account-edit"
          @click.stop="emit('edit', props.id)"
        />

        <!-- Delete icon -->
        <UButton
          v-if="loggedIn"
          class="cursor-pointer bg-red-600 text-lg text-white hover:bg-red-700"
          icon="i-mdi-delete"
          @click.stop="toggleModalDelete"
        />
      </div>
    </div>
  </UCard>

  <!-- Modal -->
  <ModalConfirm
    :open="showModalDelete"
    @close="toggleModalDelete"
    @confirm="confirmDelete"
  />
</template>
