<template>
  <footer class="relative w-full overflow-hidden bg-blue-900 py-6 text-white">
    <div
      class="pointer-events-none absolute top-0 left-1/2 z-0 w-[300px] -translate-x-1/2 opacity-10"
    >
      <img
        class="h-auto w-auto object-contain"
        src="/public/img/logo-black.png"
        alt="Logo"
      />
    </div>
    <div
      class="relative z-10 mx-auto flex max-w-4xl flex-col flex-wrap gap-2 px-4 text-center sm:flex-row sm:justify-between"
    >
      <NuxtLink to="/"
        ><UButton
          class="m-0 cursor-pointer bg-transparent p-0 text-white underline-offset-2 transition-colors hover:bg-transparent hover:text-blue-400"
          >Home</UButton
        ></NuxtLink
      >
      <NuxtLink to="/bulletin"
        ><UButton
          class="m-0 cursor-pointer bg-transparent p-0 text-white underline-offset-2 transition-colors hover:bg-transparent hover:text-blue-400"
          >Bulletin</UButton
        ></NuxtLink
      >
      <NuxtLink to="/events"
        ><UButton
          class="m-0 cursor-pointer bg-transparent p-0 text-white underline-offset-2 transition-colors hover:bg-transparent hover:text-blue-400"
          >Events</UButton
        ></NuxtLink
      >
      <NuxtLink to="/executive-board"
        ><UButton
          class="m-0 cursor-pointer bg-transparent p-0 text-white underline-offset-2 transition-colors hover:bg-transparent hover:text-blue-400"
          >Executive Board</UButton
        ></NuxtLink
      >
      <NuxtLink to="/members"
        ><UButton
          class="m-0 cursor-pointer bg-transparent p-0 text-white underline-offset-2 transition-colors hover:bg-transparent hover:text-blue-400"
          >Members</UButton
        ></NuxtLink
      >
      <NuxtLink to="/resources">
        <UButton 
        class="m-0 cursor-pointer bg-transparent p-0 text-white underline-offset-2 transition-colors hover:bg-transparent hover:text-blue-400"
        >
          Resources
        </UButton>
      </NuxtLink>
      <NuxtLink to="/history"
        ><UButton
          class="m-0 cursor-pointer bg-transparent p-0 text-white underline-offset-2 transition-colors hover:bg-transparent hover:text-blue-400"
          >History</UButton
        ></NuxtLink
      >
      <NuxtLink to="/bylaws"
        ><UButton
          class="m-0 cursor-pointer bg-transparent p-0 text-white underline-offset-2 transition-colors hover:bg-transparent hover:text-blue-400"
          >Bylaws</UButton
        ></NuxtLink
      >
      
    </div>

    <div class="z-10 mt-6 flex w-full flex-col text-center text-sm text-white">
      <div>
        &copy; Plais {{ new Date().getFullYear() }} - Website created by
      </div>
      <div class="mt-2 flex flex-row justify-center gap-3">
        <template v-for="(person, index) in displayNames" :key="index">
          <UButton
            :to="person.git"
            target="_blank"
            variant="ghost"
            class="m-0 bg-transparent p-0 text-white underline-offset-2 transition-colors hover:bg-transparent hover:text-blue-400"
          >
            {{ person.name }}
          </UButton>
        </template>
      </div>
    </div>
  </footer>
</template>

<script setup lang="ts">
interface Creator {
  name: string
  git: string
}

const creators: Creator[] = [
  {
    name: 'Wojciech Bieńkowski',
    git: 'https://github.com/vojtolini',
  },
  {
    name: 'Kacper Kazimierczuk',
    git: 'https://github.com/kaspares',
  },
  {
    name: 'Alessio Ricciardi',
    git: 'https://github.com/alessioricciardi',
  },
]
const displayNames = ref<Creator[]>([])

function shuffleNames() {
  displayNames.value = [...creators].sort(() => Math.random() - 0.5)
}

onMounted(() => {
  shuffleNames()
})
</script>
