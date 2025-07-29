<script setup lang="ts">
//types
import type { User } from '~/types'
import type { DropdownMenuItem } from '@nuxt/ui'

//Variables
const loggedIn = useState<boolean>('loggedIn')
const API_URL = useRuntimeConfig().public.API_URL
const toast = useToast()
const showSlideover = ref<boolean>(false)
const showModalBecomeMember = ref<boolean>(false)
const username = ref<string>('')

//Functions

const login = async (): Promise<void> => {
  try {
    const data: User = await $fetch(`${API_URL}/auth/me`, {
      credentials: 'include',
    })
    loggedIn.value = !!data

    toast.add({
      icon: 'mdi:account-cowboy-hat',
      title: 'Success',
      description: `Welcome back, ${data.username}!`,
      color: 'success',
    })
  } catch {
    useRouter().push('/login')
  }
}

//toggle slideover
const toggleSlideover = (): void => {
  showSlideover.value = !showSlideover.value
}

//toggle modal become a member
const toggleModalBecomeMember = (): void => {
  showModalBecomeMember.value = !showModalBecomeMember.value
}

//logout
const logout = async (): Promise<void> => {
  try {
    $fetch(`${API_URL}/auth/logout`, {
      method: 'POST',
      credentials: 'include',
    })

    loggedIn.value = false

    toast.add({
      icon: 'mdi:account-cancel',
      title: 'Success',
      description: 'Logged out successfully.',
      color: 'success',
    })
  } catch {
    useFetchError()
  }
}

//get username
watch(
  () => loggedIn.value,
  async (val) => {
    if (val) {
      try {
        const data: User = await $fetch(`${API_URL}/auth/me`, {
          credentials: 'include',
        })
        username.value = data.username
      } catch {}
    }
  },
  { immediate: true },
)

//dropdown menu
const dropdownItems = ref<DropdownMenuItem[]>([
  [
    {
      label: 'Admin Panel',
      icon: 'mdi:account-details',
      to: '/admin/panel',
    },
    {
      label: 'Logout',
      icon: 'mdi:logout',
      color: 'error',
      onSelect() {
        logout()
      },
    },
  ],
])
</script>

<template>
  <div>
    <!-- Header -->
    <div
      class="bg-gradient-to-r from-blue-800 from-20% to-blue-500 text-white shadow-lg"
    >
      <div class="flex items-center justify-between p-1.5">
        <div
          class="max-w-35 cursor-pointer"
          @click.stop="useRouter().push('/')"
        >
          <img alt="PLAIS Logo" src="/img/logo-transparent.png" />
        </div>
        <div class="text-center text-3xl font-semibold text-white lg:text-4xl">
          <p>The Polish Chapter Of Association For Information Systems</p>
        </div>
        <!-- Don't delete, necessary for proper allignment -->
        <div />
      </div>
      <div
        class="font-size flex h-20 items-center justify-between bg-white p-1.5 font-medium text-nowrap text-black"
      >
        <!-- Navigation -->
        <div>
          <!-- Mobile -->
          <UButton
            icon="mdi:menu"
            variant="link"
            color="neutral"
            class="ml-2 cursor-pointer text-3xl text-gray-900 hover:text-gray-900 lg:hidden"
            @click.stop="toggleSlideover"
          />
          <NavbarSlideover :open="showSlideover" @close="toggleSlideover" />
          <!-- Desktop -->
          <div
            class="hidden items-center gap-2 pr-5 pl-2 lg:flex xl:gap-3 xl:pl-5"
          >
            <NuxtLink to="/"><NavbarButton>Home</NavbarButton></NuxtLink>
            <NuxtLink to="/bulletin"
              ><NavbarButton>Bulletin</NavbarButton></NuxtLink
            >
            <NuxtLink to="/events"
              ><NavbarButton>Events</NavbarButton></NuxtLink
            >
            <NuxtLink to="/executive-board"
              ><NavbarButton>Executive Board</NavbarButton></NuxtLink
            >
            <NuxtLink to="/members"
              ><NavbarButton>Members</NavbarButton></NuxtLink
            >
            <NuxtLink to="/resources"
              ><NavbarButton>Resources</NavbarButton></NuxtLink
            >
            <NuxtLink to="/history"
              ><NavbarButton>History</NavbarButton></NuxtLink
            >
            <NuxtLink to="/bylaws"
              ><NavbarButton>Bylaws</NavbarButton></NuxtLink
            >
          </div>
        </div>
        <div
          v-if="!loggedIn"
          class="flex items-center gap-2 pr-2 xl:gap-5 xl:pr-5"
        >
          <!-- Login button -->
          <NavbarButton @click.stop="login()">Login</NavbarButton>

          <!-- Become a member button -->
          <UButton
            label="Become a member"
            trailing-icon="mdi:school"
            class="font-size h-10 cursor-pointer rounded-2xl bg-gradient-to-r from-blue-700 to-violet-500 px-3 hover:bg-gradient-to-r hover:from-blue-800 hover:to-blue-500 active:scale-98"
            :ui="{ trailingIcon: ' text-2xl' }"
            @click.stop="toggleModalBecomeMember"
          />
        </div>
        <div v-else>
          <!-- Logged in panel -->
          <UDropdownMenu
            :items="dropdownItems"
            :ui="{ item: 'cursor-pointer' }"
            size="xl"
            :content="{
              side: 'bottom',
              align: 'end',
            }"
          >
            <UButton
              :label="username"
              color="neutral"
              variant="ghost"
              class="cursor-pointer rounded-lg"
              :ui="{ label: 'font-size text-gray-900 underline' }"
            >
              <template #trailing>
                <div
                  class="ml-1 flex size-9 items-center justify-center rounded-full bg-gray-900"
                >
                  <UIcon name="mdi:account-badge" class="text-2xl text-white" />
                </div>
              </template>
            </UButton>
          </UDropdownMenu>
        </div>

        <!-- Become a mamber modal -->
        <NavbarBecomeMember
          :open="showModalBecomeMember"
          @close="toggleModalBecomeMember"
          @click.self="toggleModalBecomeMember"
        />
      </div>
    </div>
  </div>
</template>

<style>
.font-size {
  font-size: 17px;
}
</style>
