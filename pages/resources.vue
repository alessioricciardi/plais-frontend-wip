<script setup lang="ts">
//types
import type { Resource } from '~/types'

//variables
const resources = ref<Resource[] | null>(null)
const selectedCategory = ref<number>(-1)
const API_URL = useRuntimeConfig().public.API_URL
const loggedIn = useState('loggedIn')

//Functions

//load resources
const loadResources = async (): Promise<void> => {
  try {
    resources.value = await $fetch<Resource[]>(
      `${API_URL}/api/Resources/categories/with-details`,
    )
  } catch {
    useFetchError()
  }
}

//select category
const selectCategory = (newCategory: number): void => {
  selectedCategory.value = newCategory
}

//on Mounted
onMounted(() => {
  loadResources()
})
</script>

<template>
  <div class="w-full">
    <div class="flex">
      <div class="flex bg-gray-100/30">
        <!-- Navigation -->
        <ResourcesPanel
          v-if="resources && resources.length > 0"
          :resources="resources"
          :selected="selectedCategory"
          @change="selectCategory"
          @refresh="loadResources"
        />
      </div>
      <div class="flex-1">
        <!-- Resources header -->
        <div class="flex h-30 items-center justify-center">
          <div class="ml-5 text-5xl font-bold text-gray-900">Resources</div>
        </div>

        <div v-if="loggedIn" class="mb-5 w-full px-5">
          <ResourcesGroupAdd
            :category-id="selectedCategory"
            @refresh="loadResources"
          />
        </div>

        <!-- Links -->
        <template v-if="resources">
          <div v-for="resource in resources" :key="resource.id">
            <div v-show="resource.id === selectedCategory">
              <div class="mx-5 mb-30 flex flex-col gap-5">
                <div v-for="group in resource.groups" :key="group.id">
                  <ResourcesGroup :group="group" @refresh="loadResources" />
                </div>
              </div>
            </div>
          </div>
        </template>
      </div>
    </div>
  </div>
</template>
