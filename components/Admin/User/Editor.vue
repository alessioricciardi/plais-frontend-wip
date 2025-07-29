<script setup lang="ts">
//Types
import type { User } from '~/types'
//Props
const props = defineProps<{
  open: boolean
  isNew: boolean
}>()

//Emits
const emit = defineEmits<{
  (e: 'close'): void
  (e: 'update' | 'create', value: User): void
}>()

//Variables
const inputType = ref<string>('password')
const newUsername = ref<string>('')
const newPassword = ref<string>('')
const newPasswordConfirm = ref<string>('')
const validatorTextPassword = ref<string>('')
const validatorTextUsername = ref<string>('')
const validatorTextConfirm = ref<string>('')

//Functions

const reloadEditor = (): void => {
  newUsername.value = ''
  newPassword.value = ''
  newPasswordConfirm.value = ''
  validatorTextPassword.value = ''
  validatorTextConfirm.value = ''
}

const validatePassword = (): boolean => {
  //Check if longer than 6 characters
  if (newPassword.value.length < 6) {
    validatorTextPassword.value = 'Password must be at least 6 characters long.'
    return false
  }

  //Check if contains digits
  if (!/\d/.test(newPassword.value)) {
    validatorTextPassword.value = 'Password must contain at least 1 digit.'
    return false
  }

  //Check if contains special characters
  if (!/[!@#$%^&*(),./;'"|{}<>]/.test(newPassword.value)) {
    validatorTextPassword.value =
      'Password must contain at least 1 special character.'
    return false
  }

  //Check if contains uppercase characters
  if (!/[A-Z]/.test(newPassword.value)) {
    validatorTextPassword.value =
      'Password must contain at least 1 uppercase character.'
    return false
  }

  //Check if contains lowercase characters
  if (!/[a-z]/.test(newPassword.value)) {
    validatorTextPassword.value =
      'Password must contain at least 1 lowercase character.'
    return false
  }

  validatorTextPassword.value = ''

  //Check if passwords match
  if (newPassword.value !== newPasswordConfirm.value) {
    validatorTextConfirm.value = 'Passwords do not match.'
    return false
  }

  validatorTextConfirm.value = ''

  return true
}

//Update or create admin
const uploadAdmin = (): void => {
  if (props.isNew) {
    //Check if username contains at least 3 characters
    if (newUsername.value.length < 3) {
      validatorTextUsername.value =
        'Username must containt at least 3 characters.'
      return
    }
    validatorTextUsername.value = ''
  }

  //Validate password
  if (!validatePassword()) return

  const newAdmin: User = {
    username: newUsername.value,
    password: newPassword.value,
  }

  if (props.isNew) {
    emit('create', newAdmin)
  } else {
    emit('update', newAdmin)
  }

  emit('close')
}

// Toggle password visibility
const changeInputType = (): void => {
  inputType.value = inputType.value === 'password' ? 'text' : 'password'
}

//watches
watch(
  () => props.open,
  () => reloadEditor(),
)
</script>

<template>
  <UModal
    :title="props.isNew ? 'Admin creator' : 'Password Editor'"
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
        <template v-if="props.isNew">
          <!-- Username -->
          <UFormField
            label="Username"
            :ui="{
              label: 'text-gray-900 font-semibold',
            }"
            :error="validatorTextUsername"
          >
            <UInput
              v-model="newUsername"
              color="neutral"
              highlight
              icon="mdi:account-cowboy-hat"
              class="w-full text-lg"
            />
          </UFormField>
        </template>
        <!-- Password -->
        <UFormField
          label="New password"
          :ui="{
            label: 'text-gray-900 font-semibold',
          }"
          :error="validatorTextPassword"
        >
          <UInput
            v-model="newPassword"
            :type="inputType"
            color="neutral"
            highlight
            icon="mdi:lock"
            class="w-full text-lg"
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
              /> </template
          ></UInput>
        </UFormField>
        <!-- Confirm password -->
        <UFormField
          label="Confirm password"
          :ui="{
            label: 'text-gray-900 font-semibold ',
          }"
          :error="validatorTextConfirm"
        >
          <UInput
            v-model="newPasswordConfirm"
            type="password"
            color="neutral"
            highlight
            icon="mdi:lock"
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
          class="flex cursor-pointer bg-gray-900 hover:bg-gray-800 active:scale-99"
          @click.stop="uploadAdmin"
          >Confirm changes</UButton
        >
      </div>
    </template>
  </UModal>
</template>
