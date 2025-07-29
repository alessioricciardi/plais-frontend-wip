<script setup lang="ts">
// Types
import type { EventGroup } from '~/types'

// Props
defineProps<{
  group: EventGroup
}>()

// Emits
const emit = defineEmits<{
  (e: 'reload'): void
}>()

//State 
const loggedIn = useState<boolean>('loggedIn')

// Variables
const API_URL = useRuntimeConfig().public.API_URL
const toast = useToast()
const showModalGroupEdit = ref<boolean>(false)
const showModalEventEdit = ref<boolean>(false)
const currentEventEditId = ref<number>(-1)

// Handlers
const handleUpdate = (type: 'eventGroup' | 'event'): void => {
  if (!type) return

  if (type === 'eventGroup') {
    toggleGroupEdit()
    emit('reload')
  } else {
    closeEventEdit()
    emit('reload')
  }
}

const toggleGroupEdit = (): void => {
  showModalGroupEdit.value = !showModalGroupEdit.value
}

const openEventEdit = (id: number): void => {
  currentEventEditId.value = id
  showModalEventEdit.value = true
}

const closeEventEdit = (): void => {
  showModalEventEdit.value = false
}

const handleEventDelete = async (id: number): Promise<void> => {
  try {
    await $fetch(`${API_URL}/api/events/${id}`, {
      method: 'DELETE',
      credentials: 'include',
    })
    toast.add({
      title: 'Deleted',
      description: 'Event removed.',
      color: 'success',
    })
    emit('reload')
  } catch {
    useFetchError()
  }
}
</script>

<template>
  <UCard
    class="mb-10 w-full md:w-3xl"
    :ui="{
      header:
        'bg-gradient-to-r from-blue-800 from-20% to-blue-500 rounded-t-lg',
    }"
  >
    <!-- Header with title and edit icon -->
    <template #header>
      <div class="flex items-center justify-between">
        <div class="text-xl font-semibold text-white">
          {{ group.title }}
        </div>
        <UButton
          v-if="loggedIn"
          class="cursor-pointer bg-amber-300 text-lg text-gray-900 hover:bg-amber-400"
          icon="mdi-pencil"
          @click.stop="toggleGroupEdit"
        />
      </div>
    </template>

    <!-- Events list -->
    <div class="flex flex-col gap-2">
      <EventCard
        v-for="event in group.events"
        :id="event.id"
        :key="event.id"
        :name="event.name"
        :link="event.linkUrl"
        @edit="openEventEdit"
        @delete="handleEventDelete"
      />
    </div>

    <!-- Footer -->
    <template #footer>
      <div class="text-sm text-gray-500">
        {{ group.events.length }} event{{
          group.events.length === 1 ? '' : 's'
        }}
        in this group
      </div>
    </template>
  </UCard>

  <!-- Event Group Editor -->
  <EventGroupEditor
    :id="group.id"
    :open="showModalGroupEdit"
    @close="toggleGroupEdit"
    @update="handleUpdate('eventGroup')"
  />

  <!-- Event Editor -->
  <EventEditor
    :id="currentEventEditId"
    :open="showModalEventEdit"
    @close="closeEventEdit"
    @update="handleUpdate('event')"
  />
</template>
