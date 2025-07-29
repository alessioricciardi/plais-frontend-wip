<script setup lang="ts">
//Types
import type { EventGroupResponse, EventGroup } from '~/types'

//Variables
const API_URL = useRuntimeConfig().public.API_URL
const eventGroups = ref<EventGroup[] | null>(null)
const loggedIn = useState('loggedIn')

const currentPage = ref<number>(1)
const totalPages = ref<number>(1)
const searchPhrase = ref<string>('')

//Functions
const loadEventGroups = async () => {
  const response = await $fetch<EventGroupResponse>(
    `${API_URL}/api/eventGroups`,
    {
      params: {
        PageNumber: currentPage.value,
        PageSize: 5,
        searchPhrase: searchPhrase.value,
      },
    },
  )

  eventGroups.value = response.items
  totalPages.value = response.totalPages
}

const reloadPage = (): void => {
  currentPage.value = 1
  loadEventGroups()
}

//Nuxt methods
onMounted(() => {
  reloadPage()
})

watch([searchPhrase, currentPage], loadEventGroups)
</script>

<template>
  <div class="flex w-full max-w-full flex-col items-center gap-10">
    <div class="mb-20 text-center text-5xl font-bold text-gray-900">Events</div>

    <!--Add new EventGroup or Event-->
    <EventAddGroupOrEvent
      v-if="loggedIn"
      class="w-full md:w-3xl"
      @reload="reloadPage"
    />

    <!-- Search input -->
    <div class="flex w-full justify-center md:w-3xl">
      <UInput
        v-model="searchPhrase"
        icon="mdi:magnify"
        color="neutral"
        class="w-1/2"
        placeholder="Search event groups..."
      />
    </div>
    <template v-if="eventGroups">
      <EventGroupDisplay
        v-for="group in eventGroups"
        :key="group.id"
        :group="group"
        @reload="reloadPage"
      />
    </template>

    <!-- Pagination -->
    <div
      v-if="totalPages > 1"
      class="mt-8 flex items-center justify-center gap-3"
    >
      <UButton
        class="cursor-pointer"
        icon="mdi:arrow-left-thick"
        :disabled="currentPage === 1"
        color="neutral"
        variant="soft"
        @click="currentPage--"
      />

      <span class="font-medium text-gray-700">
        Page {{ currentPage }} of {{ totalPages }}
      </span>

      <UButton
        class="cursor-pointer"
        icon="mdi:arrow-right-thick"
        :disabled="currentPage === totalPages"
        color="neutral"
        variant="soft"
        @click="currentPage++"
      />
    </div>

    <!-- Empty state -->
    <div
      v-if="eventGroups && eventGroups.length === 0"
      class="flex justify-center text-base text-gray-600"
    >
      No events to display at the moment.
    </div>

    <!-- Loading state -->
    <div v-if="!eventGroups" class="flex justify-center">
      <progress class="progress w-100" />
    </div>
  </div>
</template>
