<script setup lang="ts">
//Types
import type { CadenceWithMembers } from '~/types'

//Variables
const API_URL = useRuntimeConfig().public.API_URL
const cadencesWithMembers = ref<CadenceWithMembers[] | null>(null)
const loggedIn = useState('loggedIn')

//Functions
const loadCadencesWithMembers = async () => {
  cadencesWithMembers.value = await $fetch(
    `${API_URL}/api/cadences/with-members`,
  )
}

const reloadPage = (): void => {
  loadCadencesWithMembers()
}

//Nuxt methods
onMounted(() => {
  reloadPage()
})
</script>

<template>
  <div class="flex w-full max-w-full flex-col items-center gap-10">
    <div class="mb-20 text-center text-5xl font-bold text-gray-900">
      Executive Board
    </div>

    <template v-if="cadencesWithMembers">
      <!-- Add new Member or Cadence -->
      <ExecutiveBoardAddCadenceOrMember
        v-if="loggedIn"
        :cadences-length="cadencesWithMembers.length"
        class="w-full md:w-2xl lg:w-4xl"
        @reload="reloadPage"
      />

      <template v-for="cadence in cadencesWithMembers" :key="cadence.id">
        <ExecutiveBoardCadenceSeparator
          :cadence-name="cadence.name"
          :cadence-position="cadence.position"
          :cadence-id="cadence.id"
          :cadences-length="cadencesWithMembers.length"
          @reload="reloadPage"
        />

        <div
          v-for="member in cadence.members"
          :key="member.executiveMemberId"
          class="w-full md:w-2xl lg:w-4xl"
        >
          <ExecutiveBoardMember
            :id="member.executiveMemberId"
            :cadence-id="cadence.id"
            :cadence-length="cadence.members.length"
            :full-name="member.fullName"
            :department="member.department"
            :email="member.email"
            :phone="member.phone"
            :about="member.about"
            :role="member.role"
            :position="member.position"
            :photo-file-name="member.photoFileName"
            @reload="reloadPage"
          />
        </div>
      </template>
    </template>
    <div class="mb-20" />

    <!-- Empty Cadence List -->
    <div
      v-if="cadencesWithMembers && cadencesWithMembers.length === 0"
      class="flex justify-center text-base"
    >
      Sorry, there's currently no executive members to see.
    </div>

    <!-- Loading -->
    <div v-if="!cadencesWithMembers" class="flex justify-center">
      <progress class="progress w-100" />
    </div>
  </div>
</template>
