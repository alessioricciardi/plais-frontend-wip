<script setup lang="ts">
//types
import type { ResourceGroup } from '~/types'

//variables
const loggedIn = useState('loggedIn')
const showModalEdit = ref<boolean>(false)

//functions

//toggle modal edit
const toggleModalEdit = (): void => {
  showModalEdit.value = !showModalEdit.value
}

//Props
defineProps<{
  group: ResourceGroup
}>()

//Emits
const emit = defineEmits<{
  (e: 'refresh'): void
}>()
</script>

<template>
  <!-- Links -->
  <div class="mx-5 flex flex-col gap-2">
    <div v-if="!loggedIn">
      <div class="mb-2 text-lg font-bold text-gray-900">
        <template v-if="group.name.endsWith(':')">
          {{ group.name }}
        </template>
        <template v-else> {{ group.name }}: </template>
      </div>
    </div>
    <div v-else>
      <UButton
        :label="group.name"
        class="mb-2 cursor-pointer rounded-xl bg-gray-900 hover:bg-gray-800"
        trailing-icon="mdi:link-edit"
        color="neutral"
        :ui="{ trailingIcon: 'text-xl ml-2' }"
        @click.stop="toggleModalEdit"
      />
    </div>

    <div v-for="item in group.items" :key="item.id">
      <ResourcesLink :link="item.url" :label="item.name" />
    </div>
  </div>

  <ResourcesGroupEditor
    :group-id="group.id"
    :open="showModalEdit"
    @refresh="emit('refresh')"
    @close="toggleModalEdit"
  />
</template>
