<script setup lang="ts">
//Props
const props = defineProps<{
  open: boolean
}>()

//Emits
const emit = defineEmits<{
  (e: 'close' | 'confirm'): void
}>()
</script>

<template>
  <UModal
    :open="props.open"
    :ui="{
      content: ' lg:max-w-1/2 max-w-full',
      header:
        'bg-gradient-to-r from-blue-900  to-violet-500  rounded-t-lg text-white text-2xl h-18',
    }"
    @update:open="
      (value) => {
        if (!value) emit('close')
      }
    "
  >
    <template #title />
    <template #header>
      <div class="w-full text-center font-bold">Become a member</div>
    </template>
    <template #body>
      <div class="w-full text-center text-lg">
        <div class="p-2">
          We cordially invite you to join the Association for Information
          Systems (AIS) - please consider becoming a member!
        </div>
        <div class="mt-5 p-5 text-base">
          <strong
            ><u
              >When joining AIS, don't forget to select PLAIS as your
              chapter.</u
            ></strong
          >
        </div>
      </div>
      <!-- Join us button -->
      <div class="flex w-full justify-center">
        <UButton
          label="Join us"
          color="neutral"
          to="https://aisnet.org/general/register_member_type.asp"
          target="_blank"
          class="mt-5 mb-15 h-8 w-1/2 justify-center rounded-3xl bg-gray-900 active:scale-98"
          :ui="{ label: 'text-base' }"
        />
      </div>
    </template>
  </UModal>
</template>
