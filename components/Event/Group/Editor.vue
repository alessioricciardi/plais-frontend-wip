<script setup lang="ts">
// Types
import type { EventGroup } from '~/types'

// Props
const props = defineProps<{
  id: number
  open: boolean
}>()

// Emit
const emit = defineEmits<{
  (e: 'close' | 'update'): void
}>()

// Variables
const API_URL = useRuntimeConfig().public.API_URL
const toast = useToast()
const showModalDelete = ref(false)
const group = reactive<EventGroup>({
  id: -1,
  title: '',
  events: [],
})

// Lifecycle
watch(
  () => props.open,
  (open) => {
    if (open && props.id > 0) reloadEditor()
  }
)

const reloadEditor = async () => {
  try {
    const data = await $fetch<EventGroup>(`${API_URL}/api/eventGroups/${props.id}`)
    Object.assign(group, data)
  } catch {
    useFetchError()
  }
}

// Handlers
const updateGroup = async () => {
  try {
    await $fetch(`${API_URL}/api/eventGroups/${group.id}`, {
      method: 'PUT',
      body: JSON.stringify(group),
      headers: { 'Content-Type': 'application/json' },
      credentials: 'include',
    })
    toast.add({ title: 'Success', description: 'Group updated.', color: 'success' })
    emit('close')
    emit('update')
  } catch {
    useFetchError()
  }
  emit('close')
}

const deleteGroup = async () => {
  try {
    await $fetch(`${API_URL}/api/eventGroups/${group.id}`, {
      method: 'DELETE',
      credentials: 'include',
    })
    toast.add({ title: 'Deleted', description: 'Group removed.', color: 'success' })

    // Reset before emit
    group.id = -1
    group.title = ''
    group.events = []
    showModalDelete.value = false

    emit('close')      
    emit('update')     
  } catch {
    useFetchError()
    emit('close')
  }
}

const toggleModalDelete = (): void => {
  showModalDelete.value = !showModalDelete.value
}
</script>


<template>
  <UModal
    title="Event Group Editor"
    :open="props.open"
    :close="false"
    :ui="{
      header:
        'bg-gradient-to-r from-blue-800 from-15% to-blue-500 rounded-t-lg',
      title: 'text-white border-b-2 border-white p-1 justify-center flex',
    }"
  >
    <template #body>
      <ModalConfirm
        :open="showModalDelete"
        @close="toggleModalDelete"
        @confirm="deleteGroup"
      />

      <UFormField
        label="Title of group"
        :ui="{ label: 'text-blue-700 font-semibold' }"
      >
        <UInput
          v-model="group.title"
          color="neutral"
          class="w-full"
          icon="i-mdi-format-title"
        />
      </UFormField>

      <UButton
        v-if="props.id"
        class="mt-5 bg-red-700 hover:bg-red-800"
        label="Delete group"
        @click.stop="toggleModalDelete"
      />
    </template>

    <template #footer>
      <div class="flex justify-between gap-3">
        <UButton class="bg-red-600" @click="emit('close')"
          >Cancel changes</UButton
        >
        <UButton class="bg-blue-700" @click="updateGroup"
          >Confirm changes</UButton
        >
      </div>
    </template>
  </UModal>
</template>
