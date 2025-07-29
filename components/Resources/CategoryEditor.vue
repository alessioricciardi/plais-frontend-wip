<script setup lang="ts">
//Types
import type { SelectMenuItem } from '@nuxt/ui'
import type { ResourceCategory } from '~/types'

//Variables
const showModalDelete: Ref<boolean> = ref<boolean>(false)
const categorySelectList = ref<SelectMenuItem[]>([])
const category: ResourceCategory = reactive({
  id: -1,
  name: '',
})
const API_URL = useRuntimeConfig().public.API_URL
const toast = useToast()

//Props
const props = defineProps<{
  open: boolean
  isNew: boolean
}>()

//Functions

//reload editor
const reloadEditor = () => {
  if (!props.isNew) loadCategories()

  Object.assign(category, {
    id: -1,
    name: '',
  })
}

//toggle delete modal
const toggleModalDelete = (): void => {
  showModalDelete.value = !showModalDelete.value
}

//load categories
const loadCategories = async (): Promise<void> => {
  try {
    //Clear old cadences
    categorySelectList.value = []

    const data = await $fetch<ResourceCategory[]>(
      `${API_URL}/api/Resources/categories`,
    )

    data.forEach((cat) => {
      //Create Select Item for Category
      const categorySelectItem = {
        label: cat.name,
        onSelect() {
          Object.assign(category, {
            id: cat.id,
            name: cat.name,
          })
        },
        icon: 'mdi:book',
      }

      categorySelectList.value.push(categorySelectItem)
    })
  } catch {
    useFetchError()
  }
}

//Upload Category
const uploadCategory = async (): Promise<void> => {
  if (!category.name.trim()) {
    toast.add({
      icon: 'mdi:book-remove-outline',
      title: 'Error',
      description: 'Category name cannot be empty.',
      color: 'error',
    })
    return
  }

  if (!props.isNew && category.id < 0) {
    toast.add({
      icon: 'mdi:book-remove-outline',
      title: 'Error',
      description: 'Please select a category before confirm.',
      color: 'error',
    })
    return
  }

  const url = props.isNew
    ? `${API_URL}/api/Resources/categories`
    : `${API_URL}/api/Resources/categories/${category.id}`
  const method = props.isNew ? 'POST' : 'PUT'
  const body = props.isNew ? { name: category.name } : category

  try {
    await $fetch(url, { method, body, credentials: 'include' })

    toast.add({
      icon: 'mdi:book-plus',
      title: 'Success',
      description: props.isNew
        ? 'Successfully added a category.'
        : 'Successfully updated a category.',
      color: 'success',
    })
  } catch {
    useFetchError()
  }

  emit('refresh')
  emit('close')
}

//delete category
const deleteCategory = async (): Promise<void> => {
  if (!props.isNew && category.id < 0) {
    toast.add({
      icon: 'mdi:book-remove-outline',
      title: 'Error',
      description: 'Please select a category before deleting.',
      color: 'error',
    })
    return
  }

  try {
    await $fetch(`${API_URL}/api/Resources/categories?id=${category.id}`, {
      method: 'DELETE',
      credentials: 'include',
    })

    toast.add({
      icon: 'mdi:book-plus',
      title: 'Success',
      description: 'Successfully deleted a category.',
      color: 'success',
    })
  } catch {
    useFetchError()
  }
  emit('refresh')
}

//Emits
const emit = defineEmits<{
  (e: 'refresh' | 'close'): void
}>()

//Watches
watch(
  () => props.open,
  () => reloadEditor(),
)
</script>

<template>
  <UModal
    :title="props.isNew ? 'Category Creator' : 'Category Editor'"
    description="Edit the necessary fields and press 'Confirm Changes' to save the changes. If you want to exit without saving, click 'Cancel changes'."
    :ui="{
      header: 'bg-gray-900 rounded-t-lg',
      title: 'text-white border-b-2 border-white p-1 justify-center flex',
      description: 'text-white mb-1',
    }"
    :open="props.open"
    :close="false"
  >
    <template #body>
      <div class="flex flex-col gap-5">
        <template v-if="!isNew">
          <!-- Choose category to configure -->
          <UFormField
            label="Select category to configure it's name"
            :ui="{ label: 'text-gray-900 font-semibold' }"
          >
            <div class="flex justify-between">
              <!-- Category Select List -->
              <USelectMenu
                :items="categorySelectList"
                color="neutral"
                placeholder="Select Category"
                icon="mdi:book-cog-outline"
                class="w-full cursor-pointer text-gray-900 ring-1 ring-[#0f172b]"
                :ui="{
                  leadingIcon: 'text-lg text-dimmed',
                  trailingIcon: 'text-gray-900',
                }"
              />
            </div>
          </UFormField>
        </template>

        <!-- Category name -->
        <UFormField
          label="Category Name"
          :ui="{
            label: 'text-gray-900 font-semibold',
          }"
        >
          <UInput
            v-model="category.name"
            color="neutral"
            highlight
            icon="mdi:book-information-variant"
            class="w-full text-lg"
          />
        </UFormField>

        <!-- Delete category -->
        <div v-if="!isNew && category.id !== -1" class="flex justify-center">
          <UButton
            label="Delete category"
            color="neutral"
            class="w-2/3 cursor-pointer justify-center bg-red-700 hover:bg-red-800 active:scale-99"
            @click.stop="toggleModalDelete"
          />
        </div>
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
          class="flex cursor-pointer bg-gray-900 hover:bg-gray-800 active:scale-99"
          @click.stop="uploadCategory"
          >Confirm changes</UButton
        >
      </div>
    </template>
  </UModal>
  <!-- Confirm member delete -->
  <ModalConfirm
    :open="showModalDelete"
    @close="toggleModalDelete"
    @confirm="(deleteCategory(), emit('close'))"
  />
</template>
