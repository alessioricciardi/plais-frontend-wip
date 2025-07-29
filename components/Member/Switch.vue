<script setup lang="ts">
//Emits
const emit = defineEmits<{
  (e: 'isCurrent' | 'isFounding'): void
}>()

//Variables
const model = ref<number>(50)
const isFounding = ref<boolean>(false)

//Functions
const toggleIsFounding = () => {
  isFounding.value = !isFounding.value
}

//Watches
watch(isFounding, (newValue, oldValue) => {
  if (newValue === oldValue) return
  emit(newValue ? 'isFounding' : 'isCurrent')
})
</script>

<template>
  <div class="flex w-full justify-center">
    <div
      class="flex flex-row items-center gap-5 text-lg font-bold sm:text-2xl md:gap-10 lg:gap-15 lg:text-3xl"
    >
      <div
        class="cursor-pointer"
        :class="{ 'text-blue-600': !isFounding }"
        @click.stop="isFounding = false"
      >
        Current Members
      </div>
      <UProgress
        v-model="model"
        :inverted="isFounding"
        class="hidden w-50 cursor-pointer md:flex"
        size="md"
        :ui="{
          indicator: 'bg-gradient-to-r from-blue-700 from-10% to-blue-600',
        }"
        @click.stop="toggleIsFounding"
      />
      <USeparator orientation="vertical" class="flex h-16 md:hidden" />
      <div
        class="cursor-pointer"
        :class="{ 'text-blue-600': isFounding }"
        @click.stop="isFounding = true"
      >
        Founding Members
      </div>
    </div>
  </div>
</template>
