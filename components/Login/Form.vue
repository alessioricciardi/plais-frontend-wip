<script setup lang="ts">
//Variables
const toast = useToast()
const API_URL = useRuntimeConfig().public.API_URL
const inputType = ref<string>('password')
const username = ref<string>('')
const password = ref<string>('')
const loginResult = ref<string>('')
const loginAttempts = ref<number>(0)
const blockLogin = ref<boolean>(false)
const blockDuration = ref<number>(0)

//Functions

// Handle logging spam
const handleLoginAttempts = (): void => {
  //Block every 4th login attempt
  if (loginAttempts.value > 2) {
    blockLogin.value = true

    //Every block adds additional 10 seconds until next login attempts
    blockDuration.value = blockDuration.value + 10000

    toast.add({
      icon: 'mdi:alert-outline',
      title: 'Too many login attempts',
      description: `Please wait ${blockDuration.value / 1000} seconds before trying again.`,
      duration: blockDuration.value,
      close: false,
      color: 'error',
      onPause: () => {},
    })

    setTimeout(() => {
      blockLogin.value = false
      loginAttempts.value = 0
    }, blockDuration.value)

    return
  }

  loginAttempts.value++
}

// Toggle password visibility
const changeInputType = (): void => {
  inputType.value = inputType.value === 'password' ? 'text' : 'password'
}

//Log in
const login = async (): Promise<void> => {
  if (!username.value || !password.value) {
    loginResult.value = 'Please enter both username and password.'
    return
  }

  if (blockLogin.value) return

  handleLoginAttempts()

  try {
    await $fetch(`${API_URL}/auth/login`, {
      method: 'POST',
      body: {
        username: username.value,
        password: password.value,
      },
      credentials: 'include',
    })

    //Clear login error
    loginResult.value = ''

    //Add loggedIn
    const loggedIn = useState<boolean>('loggedIn')
    loggedIn.value = true

    //Redirect to homepage
    useRouter().push('/executive-board')

    toast.add({
      icon: 'mdi:account-cowboy-hat',
      title: 'Success',
      description: 'Logged in successfully.',
      color: 'success',
    })
  } catch (error) {
    const err = error as FetchError
    if (err.response?.status === 401) {
      loginResult.value = 'Incorrect username or password. Please try again.'
      return
    }

    useFetchError()
  }
}
</script>

<template>
  <form @keydown.enter="login">
    <div
      class="flex h-150 w-150 flex-col items-center rounded-3xl border-1 border-gray-200 bg-white shadow-xl"
    >
      <div class="mt-15 size-40">
        <img src="/img/logo-blue.png" />
      </div>

      <div class="mt-10 flex w-2/3 flex-col gap-5">
        <UFormField
          label="Username"
          :ui="{
            label: 'ml-1 text-blue-600 text-base font-bold',
          }"
          :error="loginResult ? true : false"
        >
          <UInput
            v-model.trim="username"
            color="neutral"
            size="lg"
            highlight
            placeholder="Enter your username"
            class="w-full"
            :ui="{
              base: 'h-10 text-base rounded-lg ring-2',
            }"
          />
        </UFormField>
        <UFormField
          label="Password"
          :ui="{ label: 'ml-1 text-blue-600 text-base font-bold' }"
          :error="loginResult"
        >
          <UInput
            v-model.trim="password"
            :type="inputType"
            size="lg"
            highlight
            color="neutral"
            placeholder="Enter your password"
            class="w-full"
            :ui="{ base: 'h-10 text-base rounded-lg ring-2' }"
          >
            <template #trailing>
              <UButton
                color="neutral"
                variant="link"
                class="cursor-pointer text-xl"
                :icon="
                  inputType === 'password'
                    ? 'mdi:eye-outline'
                    : 'mdi:eye-off-outline'
                "
                @click.stop="changeInputType"
              />
            </template>
          </UInput>
        </UFormField>
        <UButton
          size="lg"
          label="Login"
          class="mt-4 w-full cursor-pointer justify-center bg-blue-600 hover:bg-blue-700 active:scale-99"
          color="neutral"
          :disabled="blockLogin"
          :ui="{ base: 'h-10 text-lg shadow-xl rounded-lg ' }"
          @click.stop="login"
        />
      </div>
    </div>
  </form>
</template>
