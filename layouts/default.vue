<script setup lang="ts">
const route = useRoute()
</script>

<template>
  <UApp>
    <div class="flex min-h-screen flex-col">
      <Navbar v-if="route.path !== '/login'" />

      <div class="flex-1">
        <div
          v-if="route.path !== '/resources' && route.path !== '/login'"
          class="mt-20"
        >
          <slot />
        </div>
        <div v-else>
          <slot />
        </div>
      </div>

      <SiteFooter v-if="route.path !== '/login'" />
    </div>
  </UApp>
</template>
