<script setup lang="ts">
//Types
import type { User } from '~/types'
import type { TableColumn, DropdownMenuItem } from '@nuxt/ui'

//Variables
const API_URL = useRuntimeConfig().public.API_URL
const users = ref<User[] | null>(null)
const selectedUser = ref<string>('')
const showModalEdit = ref<boolean>(false)
const isNewAdmin = ref<boolean>(false)
const showModalDelete = ref<boolean>(false)
const toast = useToast()
const loggedIn = useState('loggedIn')

//Functions

//load users
const loadUsers = async (): Promise<void> => {
  try {
    users.value = await $fetch(`${API_URL}/admins`, {
      credentials: 'include',
    })
  } catch {
    useFetchError()
  }
}

//create user
const createUser = async (newUser: User): Promise<void> => {
  try {
    await $fetch(`${API_URL}/admins`, {
      method: 'POST',
      body: {
        userName: newUser.username,
        password: newUser.password,
      },
      credentials: 'include',
    })

    toast.add({
      icon: 'mdi:account-check',
      title: 'Success',
      description: 'User added successfully.',
      color: 'success',
    })
  } catch {
    useFetchError()
  }
  loadUsers()
}

//update user
const updateUser = async (newUser: User): Promise<void> => {
  try {
    await $fetch(`${API_URL}/admins/${selectedUser.value}/reset-password`, {
      method: 'PUT',
      body: {
        newPassword: newUser.password,
      },
      credentials: 'include',
    })
    loadUsers()

    toast.add({
      icon: 'mdi:account-check',
      title: 'Success',
      description: 'Password changed successfully.',
      color: 'success',
    })
  } catch {
    useFetchError()
  }
}

//delete user
const deleteUser = async (): Promise<void> => {
  try {
    await $fetch(`${API_URL}/admins/${selectedUser.value}`, {
      method: 'DELETE',
      credentials: 'include',
    })
    loadUsers()

    toast.add({
      icon: 'mdi:account-check',
      title: 'Success',
      description: 'User deleted successfully.',
      color: 'success',
    })
  } catch {
    useFetchError()
  }
}

//clear unused images
const clearImages = (): void => {
  try {
    $fetch(`${API_URL}/api/Image/unused-images`, {
      method: 'DELETE',
      credentials: 'include',
    })

    toast.add({
      icon: 'mdi:image-sync-outline',
      title: 'Success',
      description: 'Images cleared successfully.',
      color: 'success',
    })
  } catch {
    useFetchError()
  }
}

//toggle modal edit
const toggleModalEdit = (): void => {
  showModalEdit.value = !showModalEdit.value
}

//toggle modal delete
const toggleModalDelete = (): void => {
  showModalDelete.value = !showModalDelete.value
}

//onMounted
onMounted(() => {
  loadUsers()
})

//Redirect to home page when not logged in
watch(
  () => loggedIn.value,
  (val) => {
    if (!val) {
      useRouter().push('/')
    } else {
      loadUsers()
    }
  },
  { immediate: true },
)

//table columns
const userColumns: TableColumn<User>[] = [
  {
    accessorKey: 'username',
    header: 'Username',
  },
  {
    id: 'actions',
    header: () =>
      h('div', { class: 'flex justify-end w-full' }, [
        h(resolveComponent('UButton'), {
          label: 'Add new user',
          trailingIcon: 'mdi:account-plus',
          class: 'cursor-pointer bg-gray-900 hover:bg-gray-700 active:scale-98',
          ui: { trailingIcon: 'text-lg', label: 'text-md' },
          onClick: () => {
            isNewAdmin.value = true
            toggleModalEdit()
          },
        }),
      ]),
  },
]

//Get actions dropdown for each row
const getDropdownActions = (user: User): DropdownMenuItem[] => {
  return [
    {
      label: 'Change password',
      icon: 'mdi:square-edit-outline',
      onClick: () => {
        selectedUser.value = user.username
        isNewAdmin.value = false
        toggleModalEdit()
      },
    },
    {
      label: 'Delete User',
      icon: 'mdi:delete-off-outline',
      color: 'error',
      onClick: () => {
        selectedUser.value = user.username
        toggleModalDelete()
      },
    },
  ]
}
</script>

<template>
  <div
    v-if="loggedIn"
    class="flex h-screen flex-row bg-radial from-gray-950 to-slate-900"
  >
    <!-- Sidebar -->
    <div class="flex h-full flex-col">
      <div class="mt-5 flex w-full justify-center">
        <img
          src="/img/logo-transparent.png"
          alt="PLAIS Logo"
          class="w-26 cursor-pointer"
          @click.stop="useRouter().push('/')"
        />
      </div>

      <!-- Links -->
      <div class="mt-10 flex w-54 flex-col gap-3">
        <UButton
          label="Home"
          to="/"
          as="nuxt-link"
          icon="mdi:home"
          variant="ghost"
          class="cursor-pointer text-white hover:bg-gray-100/5"
          :ui="{ label: 'text-lg', leadingIcon: 'text-xl' }"
        />
        <UButton
          label="Bulletin"
          to="/bulletin"
          as="nuxt-link"
          icon="mdi:book-open-page-variant"
          variant="ghost"
          class="cursor-pointer text-white hover:bg-gray-100/5"
          :ui="{ label: 'text-lg', leadingIcon: 'text-xl' }"
        />
        <UButton
          label="Executive Board"
          to="/executive-board"
          as="nuxt-link"
          icon="mdi:account-card"
          variant="ghost"
          class="cursor-pointer text-white hover:bg-gray-100/5"
          :ui="{ label: 'text-lg', leadingIcon: 'text-xl' }"
        />
        <UButton
          label="Members"
          to="/members"
          as="nuxt-link"
          icon="mdi:account-group"
          variant="ghost"
          class="cursor-pointer text-white hover:bg-gray-100/5"
          :ui="{ label: 'text-lg', leadingIcon: 'text-xl' }"
        />
        <UButton
          label="Resources"
          to="/resources"
          as="nuxt-link"
          icon="mdi:link-variant"
          variant="ghost"
          class="cursor-pointer text-white hover:bg-gray-100/5"
          :ui="{ label: 'text-lg', leadingIcon: 'text-xl' }"
        />
        <UButton
          label="History"
          to="/history"
          as="nuxt-link"
          icon="mdi:timer-sand"
          variant="ghost"
          class="cursor-pointer text-white hover:bg-gray-100/5"
          :ui="{ label: 'text-lg', leadingIcon: 'text-xl' }"
        />
        <UButton
          label="Bylaws"
          to="/bylaws"
          as="nuxt-link"
          icon="mdi:book"
          variant="ghost"
          class="cursor-pointer text-white hover:bg-gray-100/5"
          :ui="{ label: 'text-lg', leadingIcon: 'text-xl' }"
        />
      </div>
      <div class="flex h-full items-end">
        <UTooltip
          :delay-duration="0"
          text="Delete unused images that are not associated with any content."
        >
          <UButton
            label="Clear unused images"
            class="mx-4 mb-10 cursor-pointer justify-center bg-rose-700 text-white hover:bg-rose-600 active:scale-98"
            @click.stop="clearImages"
          />
        </UTooltip>
      </div>
    </div>

    <div class="flex h-full w-full flex-col">
      <!-- Header -->
      <div
        class="flex h-20 w-full items-center justify-center text-lg text-white"
      >
        Admin Panel
      </div>

      <!-- Body -->
      <div class="w-full flex-1 rounded-tl-2xl bg-gray-100">
        <!-- Table -->
        <UTable
          v-if="users"
          :columns="userColumns"
          :data="users"
          class="user-table h-full rounded-2xl border-1 border-gray-200 bg-white shadow-xl lg:m-28 lg:h-auto"
          :ui="{
            thead: 'h-16',
            th: 'text-md',
            td: 'text-gray-900',
          }"
        >
          <template #actions-cell="{ row }">
            <div class="flex w-full justify-end">
              <UDropdownMenu
                :ui="{ item: 'cursor-pointer' }"
                :items="getDropdownActions(row.original as User)"
              >
                <UButton
                  icon="mdi:account-edit"
                  color="neutral"
                  variant="ghost"
                  class="mr-1 cursor-pointer rounded-lg text-xl hover:bg-gray-900 hover:text-white"
                />
              </UDropdownMenu>
            </div> </template
        ></UTable>
      </div>
    </div>
  </div>
  <!-- Confirm member delete -->
  <ModalConfirm
    :open="showModalDelete"
    @close="toggleModalDelete"
    @confirm="deleteUser"
  />
  <!-- Editor -->
  <AdminUserEditor
    :is-new="isNewAdmin"
    :open="showModalEdit"
    @close="toggleModalEdit"
    @create="createUser"
    @update="updateUser"
  />
</template>

<style>
.user-table tr:nth-child(even) {
  background-color: #f5f6f8;
}
</style>
