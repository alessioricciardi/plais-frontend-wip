<script setup lang="ts">
// Types
import type { EventGroup, EventGroupResponse } from '~/types'
import type { SelectMenuItem } from '@nuxt/ui'

type selectMenuItemWithId = SelectMenuItem & {
  id: number
}

// Emits
const emit = defineEmits<{
  (e: 'reload'): void
}>()

// Variables
const API_URL = useRuntimeConfig().public.API_URL
const toast = useToast()

const eventGroupSelectList = ref<SelectMenuItem[]>([])
const selectedEventGroup = ref<selectMenuItemWithId | null>(null)

const showModalGroup = ref<boolean>(false)
const showModalEvent = ref<boolean>(false)

const eventGroupTitle = ref<string>('')

const event = reactive({
  eventGroupId: null as number | null,
  name: '',
  linkUrl: '',
})

// Load event groups for the dropdown
const loadGroups = async (): Promise<void> => {
  try {
    const res = await $fetch<EventGroupResponse>(`${API_URL}/api/eventGroups`)

    res.items.forEach((eventGroup: EventGroup) => {
      //Create Select Item for Event Group
      const eventGroupListItem = {
        label: eventGroup.title,
        id: eventGroup.id as number,
        icon: 'i-material-symbols-calendar-clock-rounded',
      }

      eventGroupSelectList.value.push(eventGroupListItem)
    })
  } catch (error) {
    console.error('Error while fetching groups:', error)
    useFetchError()
  }
}

// Submit new event group
const submitGroup = async (): Promise<void> => {
  if (!eventGroupTitle.value.trim()) {
    toast.add({
      title: 'Validation Error',
      description: 'Group title cannot be empty.',
      color: 'error',
    })
    return
  }

  try {
    await $fetch(`${API_URL}/api/eventGroups`, {
      method: 'POST',
      body: { title: eventGroupTitle.value },
      credentials: 'include',
    })

    toast.add({
      title: 'Success',
      description: 'Event group created successfully.',
      color: 'success',
    })

    eventGroupTitle.value = ''
    showModalGroup.value = false
    emit('reload')
  } catch {
    useFetchError()
  }
}

// Submit new event
const submitEvent = async (): Promise<void> => {
  if (!event.name.trim() || !event.linkUrl.trim()) {
    toast.add({
      title: 'Validation Error',
      description: 'All fields are required.',
      color: 'error',
    })
    return
  }

  try {
    await $fetch(`${API_URL}/api/events`, {
      method: 'POST',
      body: {
        eventGroupId: selectedEventGroup.value?.id,
        name: event.name,
        linkUrl: event.linkUrl,
      },
      credentials: 'include',
    })

    toast.add({
      title: 'Success',
      description: 'Event created successfully.',
      color: 'success',
    })

    event.eventGroupId = null
    event.name = ''
    event.linkUrl = ''
    showModalEvent.value = false
    emit('reload')
  } catch {
    useFetchError()
  }
}

// Watch modal open to preload groups
watch(showModalEvent, async (open) => {
  if (open) await loadGroups()
})
</script>

<template>
  <div class="flex w-full gap-5">
    <!-- Add new Event -->
    <div
      class="flex h-24 w-1/2 cursor-pointer items-center justify-center rounded-lg border border-gray-200 bg-gray-200/60 text-gray-900 shadow-xl transition-colors hover:bg-gradient-to-r hover:from-blue-800 hover:to-blue-500 hover:text-white active:scale-99"
      @click.stop="showModalEvent = true"
    >
      Add new event
      <UIcon name="mdi:calendar-plus" size="22" class="ml-2" />
    </div>

    <!-- Add new Group -->
    <div
      class="flex h-24 w-1/2 cursor-pointer items-center justify-center rounded-lg border border-gray-200 bg-gray-200/60 text-gray-900 shadow-xl transition-colors hover:bg-gradient-to-r hover:from-blue-800 hover:to-blue-500 hover:text-white active:scale-99"
      @click.stop="showModalGroup = true"
    >
      Add new group
      <UIcon name="mdi:folder-plus" size="22" class="ml-2" />
    </div>

    <!-- Modal: Add Event -->
    <UModal
      :close="false"
      title="Create new Event"
      :open="showModalEvent"
      :ui="{
        header:
          'bg-gradient-to-r from-blue-800 from-15% to-blue-500 rounded-t-lg',
        title: 'text-white p-1 justify-center flex',
      }"
      @close="showModalEvent = false"
    >
      <template #body>
        <div class="flex flex-col gap-4">
          <UFormField label="Select Group">
            <USelectMenu
              v-model="selectedEventGroup"
              :items="eventGroupSelectList"
              color="neutral"
              placeholder="Choose group"
              class="w-full"
            >
              <template #empty>
                <div class="px-3 py-2 text-sm text-gray-400">
                  No groups available
                </div>
              </template>
            </USelectMenu>
          </UFormField>

          <UFormField label="Event Name">
            <UInput v-model="event.name" color="neutral" class="w-full" />
          </UFormField>

          <UFormField label="Link URL">
            <UInput v-model="event.linkUrl" color="neutral" class="w-full" />
          </UFormField>
        </div>
      </template>

      <template #footer>
        <div class="flex justify-end gap-4">
          <UButton
            color="error"
            variant="solid"
            class="cursor-pointer active:scale-120"
            @click="showModalEvent = false"
          >
            Cancel
          </UButton>
          <UButton
            color="primary"
            variant="solid"
            class="cursor-pointer active:scale-95"
            @click="submitEvent"
          >
            Create
          </UButton>
        </div>
      </template>
    </UModal>

    <!-- Modal: Add Group -->
    <UModal
      :close="false"
      title="Create new Event Group"
      :open="showModalGroup"
      :closable="true"
      :ui="{
        header:
          'bg-gradient-to-r from-blue-800 from-15% to-blue-500 rounded-t-lg',
        title: 'text-white p-1 justify-center flex',
      }"
      @close="showModalGroup = false"
    >
      <template #body>
        <UFormField label="Group Title (e.g. 2025)">
          <UInput v-model="eventGroupTitle" color="neutral" class="w-full" />
        </UFormField>
      </template>

      <template #footer>
        <div class="flex justify-end gap-4">
          <UButton
            color="error"
            variant="solid"
            class="cursor-pointer active:scale-95"
            @click="showModalGroup = false"
          >
            Cancel
          </UButton>
          <UButton
            color="primary"
            variant="solid"
            class="cursor-pointer active:scale-95"
            @click="submitGroup"
          >
            Create
          </UButton>
        </div>
      </template>
    </UModal>
  </div>
</template>
