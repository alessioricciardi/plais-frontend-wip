<script setup lang="ts">
//Variables
const API_URL = useRuntimeConfig().public.API_URL
const toast = useToast()

//Functions
const addNewGroup = async (): Promise<void> => {
  try {
    if (props.categoryId < 0) {
      useFetchError()
      return
    }

    await $fetch(
      `${API_URL}/api/Resources/categories/${props.categoryId}/groups`,
      {
        method: 'POST',
        body: {
          name: 'New Group',
        },
        credentials: 'include',
      },
    )
  } catch {
    useFetchError()
  }

  toast.add({
    icon: 'mdi:format-list-group-plus',
    title: 'Success',
    description:
      'Successfully added a new Group. Scroll down the page to edit a newly added Group.',
    color: 'success',
  })

  emit('refresh')
}

//Props
const props = defineProps<{
  categoryId: number
}>()

//Emits

const emit = defineEmits<{
  (e: 'refresh'): void
}>()
</script>

<template>
  <UButton
    label="Add New Group"
    class="flex w-full cursor-pointer justify-center rounded-lg border-1 border-gray-500 bg-gray-200/60 text-gray-900 shadow-xl transition-colors duration-200 ease-out hover:bg-gradient-to-r hover:from-blue-800 hover:from-20% hover:to-blue-500 hover:text-white active:scale-99"
    color="neutral"
    @click.stop="addNewGroup"
  />
</template>
