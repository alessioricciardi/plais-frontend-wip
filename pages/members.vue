<script setup lang="ts">
//Variables
const isFounding = ref<boolean>(false)

onMounted(() => {
  isFounding.value = false
})
</script>

<template>
  <div class="mt-30">
    <MemberSwitch
      @is-founding="isFounding = true"
      @is-current="isFounding = false"
    />
    <div class="mt-30">
      <MemberTable v-show="isFounding" class="lg:mx-50" :is-founding="true" />
      <MemberTable v-show="!isFounding" class="lg:mx-50" :is-founding="false" />
    </div>
  </div>
</template>
