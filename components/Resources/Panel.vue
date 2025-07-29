<script setup lang="ts">
//types
import type { TabsItem } from '@nuxt/ui'
import type { Resource } from '~/types'

//variables
const selectedCategory = ref<number>(-1)
const categories = ref<TabsItem[] | undefined>(undefined)
const loggedIn = useState('loggedIn')
const showModalEdit = ref<boolean>(false)
const isNewCategory = ref<boolean>(false)

//functions

//toggle modal edit
const toggleModalEdit = (): void => {
  showModalEdit.value = !showModalEdit.value
}

//load categories
const loadCategories = (): void => {
  if (props.resources === null || props.resources.length < 1) return

  categories.value = props.resources.map((category) => ({
    label: category.name,
    value: category.id,
    icon: 'mdi:rhombus-split',
  }))

  if (props.selected > 0) {
    selectedCategory.value = props.selected
  } else {
    selectedCategory.value = props.resources[0].id
  }
}

//props
const props = defineProps<{
  resources: Resource[]
  selected: number
}>()

//emits
const emit = defineEmits<{
  (e: 'change', selectedCategory: number): void
  (e: 'refresh'): void
}>()

//watches

//emit selected category
watch(selectedCategory, (newVal, oldVal) => {
  if (newVal !== oldVal) emit('change', selectedCategory.value)
})

//load categories
watch(
  () => props.resources,
  (newVal, oldVal) => {
    if (newVal !== oldVal) loadCategories()
  },
  {
    immediate: true,
  },
)
</script>

<template>
  <!-- Navigation -->
  <div class="h-full w-55 bg-gray-50">
    <div class="sticky top-0 mt-5 overflow-y-auto">
      <div v-if="loggedIn" class="my-5 mr-5 ml-1 flex gap-2">
        <UButton
          label="Add"
          icon="mdi:book-plus-outline"
          color="neutral"
          class="w-1/2 cursor-pointer justify-center text-base active:scale-98"
          @click.stop="((isNewCategory = true), toggleModalEdit())"
        />
        <UButton
          label="Edit"
          icon="mdi:book-edit-outline"
          color="neutral"
          class="w-1/2 cursor-pointer justify-center text-base active:scale-98"
          @click.stop="((isNewCategory = false), toggleModalEdit())"
        />
      </div>
      <UTabs
        v-model="selectedCategory"
        :items="categories"
        orientation="vertical"
        color="neutral"
        class=""
        :ui="{
          label: 'text-2xl cursor-pointer',
          indicator: 'bg-gray-900',
          list: 'bg-gray-50',
        }"
      />
    </div>
  </div>
  <USeparator orientation="vertical" size="sm" />

  <!-- Category Add/Edit -->
  <ResourcesCategoryEditor
    :open="showModalEdit"
    :is-new="isNewCategory"
    @close="toggleModalEdit"
    @refresh="emit('refresh')"
  />
</template>
