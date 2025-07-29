<script setup lang="ts">
//Types
import type { Member } from '~/types'

//Props
const props = defineProps<{
  open: boolean
  memberId?: number
  isNewMember: boolean
  isFounding: boolean
}>()

//Emits
const emit = defineEmits<{
  (e: 'close' | 'delete'): void
  (e: 'update' | 'create', value: Member): void
}>()

//Variables
const API_URL = useRuntimeConfig().public.API_URL
const newMember: Member = reactive({
  id: -1,
  fullName: '',
  email: '',
  university: '',
})

//Functions

const loadMember = async (): Promise<void> => {
  let url = null
  if (props.isFounding) {
    url = `${API_URL}/api/foundingMembers/${props.memberId}`
  } else {
    url = `${API_URL}/api/currentMembers/${props.memberId}`
  }

  if (url) {
    try {
      const data: Member = await $fetch(url)
      Object.assign(newMember, data)
    } catch {
      useFetchError()
    }
  }
}

//Reload Editor
const reloadEditor = (): void => {
  //Don't fetch when new member is being created
  if (
    !props.isNewMember &&
    props.memberId !== undefined &&
    props.memberId >= 0
  ) {
    loadMember()
  } else {
    Object.assign(newMember, {
      id: -1,
      fullName: '',
      email: '',
      university: '',
    })
  }
}

//Update or create new member
const uploadMember = (): void => {
  if (props.isNewMember) {
    emit('create', { ...newMember })
    emit('close')
    return
  }

  emit('update', { ...newMember })
  emit('close')
}

//onMounted
onMounted(() => {
  reloadEditor()
})

//Watches
watch(
  () => props.open,
  () => {
    reloadEditor()
  },
)
</script>

<template>
  <UModal
    title="Member Editor"
    description="Edit the necessary fields and press 'Confirm Changes' to save the changes. If you want to exit without saving, click 'Cancel changes'."
    :ui="{
      header:
        'bg-gradient-to-r from-blue-800 from-15% to-blue-500 rounded-t-lg',
      title: 'text-white border-b-2 border-white p-1 justify-center flex',
      description: 'text-white mb-1',
    }"
    :open="props.open"
    :close="false"
  >
    <template #body>
      <div
        v-if="isNewMember || (!isNewMember && newMember.id !== -1)"
        class="flex flex-col gap-5"
      >
        <!-- Member name -->
        <UFormField
          label="Full Name"
          :ui="{ label: 'text-blue-700 font-semibold' }"
        >
          <UInput
            v-model="newMember.fullName"
            color="neutral"
            highlight
            icon="i-mdi-account-file-text"
            class="w-full text-lg"
          />
        </UFormField>

        <!-- Email -->
        <UFormField
          label="Email"
          :ui="{ label: 'text-blue-700 font-semibold' }"
        >
          <UInput
            v-model="newMember.email"
            color="neutral"
            highlight
            icon="i-mdi-email"
            class="w-full text-lg"
          />
        </UFormField>

        <!-- University -->
        <UFormField
          label="University"
          :ui="{ label: 'text-blue-700 font-semibold' }"
        >
          <UInput
            v-model="newMember.university"
            color="neutral"
            highlight
            icon="mdi:school"
            class="w-full text-lg"
          />
        </UFormField>
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
          @click.stop="uploadMember"
          >Confirm changes</UButton
        >
      </div>
    </template>
  </UModal>
</template>
