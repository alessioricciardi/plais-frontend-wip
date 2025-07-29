<script setup lang="ts">
//Types
import type { Member } from '~/types'
import type { TableColumn, DropdownMenuItem } from '@nuxt/ui'
//Props
const props = defineProps<{
  isFounding: boolean
}>()

//Variables
const loggedIn = useState('loggedIn')
const API_URL = useRuntimeConfig().public.API_URL
const members = ref<Member[] | null>(null)
const toast = useToast()

const showModalDelete = ref<boolean>(false)
const showModalEdit = ref<boolean>(false)
const selectedMemberId = ref<number>(-1)
const isNewMember = ref<boolean>(false)

//onMounted
onMounted(() => {
  loadMembers()
})

//Functions

//Load members
const loadMembers = async (): Promise<void> => {
  let url = null

  if (props.isFounding) {
    url = `${API_URL}/api/foundingMembers`
  } else {
    url = `${API_URL}/api/currentMembers`
  }

  if (url) {
    try {
      members.value = await $fetch(url)
    } catch {
      useFetchError()
    }
  }
}

//Create member
const createMember = async (newMember: Member): Promise<void> => {
  let url = null

  if (props.isFounding) {
    url = `${API_URL}/api/foundingMembers/`
  } else {
    url = `${API_URL}/api/currentMembers/`
  }

  try {
    await $fetch(url, {
      method: 'POST',
      body: newMember,
      credentials: 'include',
    })

    toast.add({
      icon: 'mdi:account-check',
      title: 'Success',
      description: 'Member created successfully.',
      color: 'success',
    })
    loadMembers()
  } catch {
    useFetchError()
  }
}

//Update member
const updateMember = async (newMember: Member): Promise<void> => {
  let url = null

  if (props.isFounding) {
    url = `${API_URL}/api/foundingMembers/${newMember.id}`
  } else {
    url = `${API_URL}/api/currentMembers/${newMember.id}`
  }

  try {
    await $fetch(url, {
      method: 'PUT',
      body: newMember,
      credentials: 'include',
    })

    toast.add({
      icon: 'mdi:account-check',
      title: 'Success',
      description: 'Member updated successfully.',
      color: 'success',
    })
    loadMembers()
  } catch {
    useFetchError()
  }
}

//Delete member
const deleteMember = async (): Promise<void> => {
  if (selectedMemberId.value < 0) return

  let url = null

  if (props.isFounding) {
    url = `${API_URL}/api/foundingMembers/${selectedMemberId.value}`
  } else {
    url = `${API_URL}/api/currentMembers/${selectedMemberId.value}`
  }

  try {
    await $fetch(url, {
      method: 'DELETE',
      credentials: 'include',
    })

    toast.add({
      icon: 'mdi:account-cancel',
      title: 'Success',
      description: 'Member deleted successfully.',
      color: 'success',
    })

    loadMembers()
  } catch {
    useFetchError()
  }
}

//Toggle modal delete
const toggleModalDelete = (): void => {
  showModalDelete.value = !showModalDelete.value
}

//Toggle modal edit
const toggleModalEdit = (): void => {
  showModalEdit.value = !showModalEdit.value
}

const addNewMember = (): void => {
  isNewMember.value = true
  toggleModalEdit()
}

//Get actions dropdown for each row
const getDropdownActions = (member: Member): DropdownMenuItem[] => {
  return [
    {
      label: 'Edit Member',
      icon: 'mdi:square-edit-outline',
      onClick: () => {
        selectedMemberId.value = member.id
        isNewMember.value = false
        toggleModalEdit()
      },
    },
    {
      label: 'Delete Member',
      icon: 'mdi:delete-off-outline',
      color: 'error',
      onClick: () => {
        selectedMemberId.value = member.id
        toggleModalDelete()
      },
    },
  ]
}

//member table columns
const MemberColumns: TableColumn<Member>[] = [
  {
    accessorKey: 'fullName',
    header: 'Full Name',
  },
  {
    accessorKey: 'email',
    header: 'E-mail',
  },
  {
    accessorKey: 'university',
    header: 'University',
  },
  {
    id: 'actions',
  },
]
</script>

<template>
  <div>
    <div v-if="members">
      <div class="flex w-full justify-center">
        <!-- Add new member -->
        <div
          v-if="loggedIn"
          class="mb-10 flex h-16 w-1/2 cursor-pointer items-center justify-center gap-2 rounded-lg border border-gray-200 bg-gray-200/60 text-gray-900 shadow-xl transition-colors duration-200 ease-out hover:bg-gradient-to-r hover:from-blue-800 hover:from-20% hover:to-blue-500 hover:text-white active:scale-99"
          @click.stop="addNewMember"
        >
          Add new member
          <UIcon name="mdi:account-plus" size="22" />
        </div>
      </div>
      <div class="member-table m-1 rounded-xl border-1 border-gray-200">
        <!-- Confirm member delete -->
        <ModalConfirm
          :open="showModalDelete"
          @close="toggleModalDelete"
          @confirm="deleteMember"
        />

        <!-- Editing mode -->
        <MemberEditor
          :member-id="selectedMemberId"
          :is-new-member="isNewMember"
          :is-founding="isFounding"
          :open="showModalEdit"
          @close="toggleModalEdit"
          @update="updateMember"
          @create="createMember"
        />

        <UTable
          class="rounded-xl"
          :columns="MemberColumns"
          :data="members"
          empty="Sorry, there's currently no  members to see."
          :ui="{
            thead: 'bg-gradient-to-r from-blue-800 from-60% to-blue-600',
            th: 'text-white text-base lg:text-lg',
            td: 'text-gray-900 text-base',
            empty: 'text-gray-900',
          }"
        >
          <template v-if="loggedIn" #actions-cell="{ row }">
            <div class="flex w-full justify-end">
              <UDropdownMenu
                class=""
                :ui="{ item: 'cursor-pointer' }"
                :items="getDropdownActions(row.original as Member)"
              >
                <UButton
                  icon="mdi:account-edit"
                  color="neutral"
                  variant="ghost"
                  class="mr-1 cursor-pointer rounded-lg text-xl hover:bg-gray-900 hover:text-white"
                />
              </UDropdownMenu>
            </div>
          </template>
        </UTable>
      </div>
    </div>
    <!-- Loading -->
    <div v-if="!members" class="flex justify-center">
      <progress class="progress w-100" />
    </div>
  </div>
</template>

<style>
.member-table tr:nth-child(even) {
  background-color: #f5f6f8;
}
</style>
