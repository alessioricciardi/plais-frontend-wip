<script setup lang="ts">
//Types
import type { Event } from '~/types'

//Props
const props = defineProps<{
  id: number
  open: boolean
}>()

//Emit
const emit = defineEmits<{
  (e: 'close' | 'delete'): void
  (e: 'update', value: Event): void
}>()

//Variables
const API_URL = useRuntimeConfig().public.API_URL
const toast = useToast()
const showModalDelete = ref(false)
const event = reactive<Event>({
  id: -1,
  name: '',
  linkUrl: '',
  eventGroupId: -1,
})

//Watch
watch(
  () => props.open,
  async (open) => {
    if (open && props.id !== -1) {
      await reloadEditor()
    }
  },
  { immediate: true }
)

//Reload
const reloadEditor = async () => {
  try {
    const data = await $fetch<Event>(`${API_URL}/api/events/${props.id}`)
    Object.assign(event, data)
  } catch {
    useFetchError()
  }
}

//Handlers
const updateEvent = async () => {
  try {
    await $fetch(`${API_URL}/api/events/${event.id}`, {
      method: 'PUT',
      body: JSON.stringify(event),
      headers: { 'Content-Type': 'application/json' },
      credentials: 'include',
    })
    toast.add({ title: 'Success', description: 'Event updated.', color: 'success' })
    emit('update', event)
    emit('close')
  } catch {
    useFetchError()
  }
}
const toggleModalDelete = () => showModalDelete.value = !showModalDelete.value
</script>


<template>
  <UModal
    title="Event Editor"
    :open="props.open"
    :close="false"
    :ui="{
      header: 'bg-gradient-to-r from-blue-800 from-15% to-blue-500 rounded-t-lg',
      title: 'text-white border-b-2 border-white p-1 justify-center flex',
    }"
  >
    <template #body>
      <UFormField label="Event Name" :ui="{ label: 'text-blue-700 font-semibold' }">
        <UInput v-model="event.name" icon="i-mdi-text" />
      </UFormField>

      <UFormField label="Link URL" class="mt-2" :ui="{ label: 'text-blue-700 font-semibold' }">
        <UInput v-model="event.linkUrl" icon="i-mdi-link" />
      </UFormField>
    </template>

    <template #footer>
      <div class="flex justify-between gap-3">
        <UButton class="bg-red-600" @click="emit('close')">Cancel changes</UButton>
        <UButton class="bg-blue-700" @click="updateEvent">Confirm changes</UButton>
      </div>
    </template>
  </UModal>
</template>
