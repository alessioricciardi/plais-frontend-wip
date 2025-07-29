<script setup lang="ts">
import DOMPurify from 'dompurify'
//emits
const emit = defineEmits(['edit', 'delete'])
//props
const props = defineProps<{
  id: number
  title: string
  content: string
  dateCreated?: string
  photos?: string[]
}>()
//clean content
const sanitizedContent = computed(() => DOMPurify.sanitize(props.content))
//date format 
const formattedDate = computed(() => {
  if (!props.dateCreated) return ''
  const date = new Date(props.dateCreated)
  return date.toLocaleDateString('en-GB', {
    year: 'numeric',
    month: 'long',
    day: 'numeric',
  })
})
//log state
const loggedIn = useState('loggedIn')

onMounted(() => {
  const wrappers = document.querySelectorAll('.v-html-wrapper')
  
  wrappers.forEach((wrapper) => {
    wrapper.querySelectorAll('table').forEach((table) => {
      const parent = table.parentElement
      if (!parent?.classList.contains('table-wrapper')) {
        const container = document.createElement('div')
        container.className = 'table-wrapper'
        table.replaceWith(container)
        container.appendChild(table)
      }
    })
  })
})


</script>

<template>
  <div
    class="relative bottom-5 mx-auto w-[80%] rounded-lg bg-gray-50 p-4 shadow-lg mt-10"
  >
    <!--Logo Plais-->
    <div class="relative h-auto w-auto">
      <img
        class="block w-1/2 sm:w-1/4 md:w-1/6 lg:w-1/12 max-w-[64px] object-contain mx-auto mb-4 lg:absolute
               "
        src="/public/img/logo-black.png"
        alt="Logo"
      >
    </div>

    <h2
      class="mb-2 w-full text-center font-bold break-words text-black sm:text-2xl md:text-4xl"
    >
      {{ title }}
    </h2>
    <p v-if="dateCreated" class="text-center text-sm text-gray-500">
      {{ formattedDate }}
    </p>
    <div class="prose v-html-wrapper max-w-none" v-html="sanitizedContent" />
    <!--Edit emit-->
    <div v-if="loggedIn" class="absolute top-2 right-2 flex gap-2">
      <UButton
        icon="material-symbols:edit"
        class="cursor-pointer bg-blue-500 text-lg text-gray-900 hover:bg-blue-600"
        @click="emit('edit', id)"
      />
      <!--Delete emit-->
      <UButton
        icon="material-symbols:delete-forever-rounded"
        class="cursor-pointer bg-red-500 text-lg text-gray-900 hover:bg-red-600"
        @click="emit('delete', id)"
      />
    </div>
  </div>
</template>

<style>
.v-html-wrapper {
  max-width: 100%;
  width: 100%;
  overflow-wrap: break-word;
  word-break: break-word;
}

/* Obrazy */
.v-html-wrapper img {
  max-width: 100%;
  height: auto;
  object-fit: contain;
  margin: auto;
  border: 1px solid #e5e7eb;
  border-radius: 12px;
  box-shadow: 0 10px 25px rgba(0, 0, 0, 0.1);
  transition: transform 0.3s ease, box-shadow 0.3s ease;
  display: block;
}

.v-html-wrapper img.align-center {
  margin-left: auto;
  margin-right: auto;
}

/* Tabele w oddzielnym wrapperze */
.v-html-wrapper .table-wrapper {
  overflow-x: auto;
  width: 100%;
}

/* Sama tabela nie wymusza przewijania wrappera */
.v-html-wrapper table {
  border-collapse: collapse;
  table-layout: auto;
  width: 100%;
}

.v-html-wrapper th,
.v-html-wrapper td {
  border: 1px solid #000000;
  padding: 0.5rem;
  vertical-align: top;
  box-sizing: border-box;
  white-space: normal;
}

/* Styl nagłówków */
.v-html-wrapper thead {
  background-color: #f3f4f6;
  font-weight: 600;
}

</style>