<script setup lang="ts">
//Types
import type { ResourceGroup } from '~/types'

//Variables
const showModalDelete: Ref<boolean> = ref<boolean>(false)
const group: ResourceGroup = reactive({
  id: -1,
  name: '',
  items: [],
})
const API_URL = useRuntimeConfig().public.API_URL
const toast = useToast()

//Props
const props = defineProps<{
  groupId: number
  open: boolean
}>()

//Functions

//Toggle delete modal
const toggleModalDelete = (): void => {
  showModalDelete.value = !showModalDelete.value
}

//Load group
const loadGroup = async (): Promise<void> => {
  try {
    const data = await $fetch<ResourceGroup>(
      `${API_URL}/api/Resources/groups/${props.groupId}/details`,
    )

    Object.assign(group, data)
  } catch {}
}

//Upload Group
const uploadGroup = async (): Promise<void> => {
  try {
    await $fetch(`${API_URL}/api/Resources/groups/${group.id}/with-items`, {
      method: 'PUT',
      body: group,
      credentials: 'include',
    })

    toast.add({
      icon: 'mdi:format-list-group-plus',
      title: 'Success',
      description: 'Successfully updated a group.',
      color: 'success',
    })
  } catch {
    useFetchError()
  }

  emit('refresh')
  emit('close')
}

const deleteGroup = async (): Promise<void> => {
  try {
    await $fetch(`${API_URL}/api/Resources/groups/${group.id}`, {
      method: 'DELETE',
      credentials: 'include',
    })

    toast.add({
      icon: 'mdi:delete-forever',
      title: 'Success',
      description: 'Successfully deleted a group.',
      color: 'success',
    })
  } catch {
    useFetchError()
  }

  emit('refresh')
  emit('close')
}

//Add new Link
const addNewLink = (): void => {
  group.items.unshift({
    name: 'Please provide a label of a link.',
    url: 'Please provide a url of a link.',
  })

  toast.add({
    icon: 'mdi:link-variant-plus',
    title: 'Success',
    description:
      'Successfully added a new Link. Check Link 1 to configure it now.',
    color: 'success',
  })
}

//Delete link
const deleteLink = (index: number): void => {
  group.items.splice(index, 1)

  toast.add({
    icon: 'mdi:link-variant-off',
    title: 'Success',
    description: 'Successfully deleted a Link.',
    color: 'success',
  })
}

//watches

//Emits
const emit = defineEmits<{
  (e: 'refresh' | 'close'): void
}>()

//Watches
watch(
  () => props.open,
  () => loadGroup(),
)
</script>

<template>
  <UModal
    title="Group Editor"
    description="Edit the necessary fields and press 'Confirm Changes' to save the changes. If you want to exit without saving, click 'Cancel changes'."
    :ui="{
      header:
        'bg-gradient-to-r from-blue-800 from-15% to-blue-500 rounded-t-lg min-h-40',
      title: 'text-white border-b-2 border-white p-1 justify-center flex',
      description: 'text-white mb-1',
    }"
    class="md:min-w-200"
    :open="props.open"
    :close="false"
  >
    <template #body>
      <div class="flex flex-col gap-10">
        <!-- Group name -->
        <UFormField
          label="Group name"
          size="lg"
          :ui="{
            label: 'text-blue-600 font-semibold',
          }"
          help="You don't need to type ' : ' (colon) at the end of the group name — it will be added automatically."
        >
          <UInput
            v-model="group.name"
            color="neutral"
            highlight
            icon="mdi:format-list-group"
            size="lg"
            class="w-full text-lg"
            :ui="{ base: 'ring-blue-500 ring-2' }"
          />
        </UFormField>

        <USeparator
          label="Links configuration"
          size="xs"
          class="mt-5"
          :ui="{
            border: 'border-gray-900',
          }"
        />

        <!-- Add new Link -->
        <UButton
          label="Add new Link"
          class="flex cursor-pointer justify-center rounded-lg border-1 border-gray-500 bg-gray-200/60 text-gray-900 shadow-xl transition-colors duration-200 ease-out hover:bg-gradient-to-r hover:from-blue-800 hover:from-20% hover:to-blue-500 hover:text-white active:scale-99"
          color="neutral"
          @click.stop="addNewLink()"
        />

        <template v-if="group.items.length > 0">
          <div
            v-for="(gr, index) in group.items"
            :key="index"
            class="flex flex-col gap-5 rounded-2xl border-2 border-blue-500 px-3 py-5 shadow-xl"
          >
            <div class="flex w-full justify-between">
              <div>Link {{ index + 1 }}</div>
              <div>
                <UTooltip
                  text="Delete this link"
                  delay="0"
                  arrow
                  :content="{ side: 'left' }"
                >
                  <UButton
                    icon="mdi:link-variant-remove"
                    class="cursor-pointer rounded-lg text-lg"
                    color="neutral"
                    @click.stop="() => deleteLink(index)"
                  />
                </UTooltip>
              </div>
            </div>

            <!-- Link label -->
            <UFormField
              label="Link label"
              size="lg"
              :ui="{
                label: 'text-gray-900 font-semibold ',
              }"
            >
              <UInput
                v-model="gr.name"
                highlight
                size="lg"
                color="neutral"
                icon="mdi:link-edit"
                class="w-full text-lg"
                :ui="{ base: 'ring-gray-400' }"
              />
            </UFormField>

            <!-- Link Url -->
            <UFormField
              label="Link url"
              size="lg"
              :ui="{
                label: 'text-gray-900 font-semibold',
              }"
            >
              <UInput
                v-model="gr.url"
                color="neutral"
                highlight
                size="lg"
                icon="mdi:link-variant"
                class="w-full text-lg"
                :ui="{ base: 'ring-gray-400' }"
              />
            </UFormField>
          </div>
        </template>
        <template v-else>
          <div class="w-full text-center">
            There's no links to be displayed.
          </div>
        </template>
      </div>

      <!-- Delete Group -->
      <div class="mt-10 mb-5 flex justify-center">
        <UButton
          label="Delete group"
          color="neutral"
          class="w-2/3 cursor-pointer justify-center bg-red-700 hover:bg-red-800 active:scale-99 md:w-1/2"
          @click.stop="toggleModalDelete"
        />
      </div>
    </template>
    <!-- Confirm / Cancel changes -->
    <template #footer>
      <div class="flex w-full justify-between">
        <UButton
          class="flex cursor-pointer bg-red-700 hover:bg-red-700/80 active:scale-99"
          @click.stop="emit('close')"
          >Cancel changes</UButton
        >
        <UButton
          class="flex cursor-pointer bg-blue-700 hover:bg-blue-700/80 active:scale-99"
          @click.stop="uploadGroup"
          >Confirm changes</UButton
        >
      </div>
    </template>
  </UModal>
  <!-- Confirm member delete -->
  <ModalConfirm
    :open="showModalDelete"
    @close="toggleModalDelete"
    @confirm="deleteGroup"
  />
</template>
