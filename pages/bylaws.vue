<script setup lang="ts">
import type { Data, response } from '~/types'
const loggedIn = useState('loggedIn')
const showEditor = ref<boolean>(false)
const isEditing = ref<boolean>(false)
const bylawsHtml = ref<string>('')
const API_URL = useRuntimeConfig().public.API_URL
const toast = useToast()

const content = ref<Data>({
  html: '',
})

const originalContent = ref<Data>({ html: '' })

onMounted(async () => {
  try {
    const res = await $fetch<response>(`${API_URL}/api/bylaws`, {
      method: 'GET',
      credentials: 'include',
    })
    bylawsHtml.value = res.content
  } catch {
    toast.add({
      title: 'Error',
      description: 'Failed to load Bylaws from server',
      color: 'error',
    })
  }
})

function startEditing() {
  content.value = {
    html: bylawsHtml.value,
  }
  originalContent.value = { ...content.value }
  isEditing.value = true
  showEditor.value = true
}

function toggleEditor() {
  showEditor.value = !showEditor.value
  if (!showEditor.value && !isEditing.value) {
    resetForm()
  }
}

function cancelEditing() {
  content.value = { ...originalContent.value }
  isEditing.value = false
  showEditor.value = false
}

function resetForm() {
  content.value = { html: '' }
  isEditing.value = false
}

async function saveChanges() {
  const payload = {
    content: content.value.html,
  }

  try {
    await $fetch<response>(`${API_URL}/api/bylaws`, {
      method: 'PUT',
      body: payload,
      credentials: 'include',
    })
    bylawsHtml.value = content.value.html
    originalContent.value = { ...content.value }
    isEditing.value = false
    showEditor.value = false
  } catch {
    toast.add({
      title: 'Error',
      description: 'Failed to update Bylaws',
      color: 'error',
    })
  }
}
</script>

<template>
  <div>
    <div class="mb-20 text-center text-5xl font-bold text-gray-900">Bylaws</div>
    <div v-if="loggedIn" class="mb-6 flex items-center justify-center">
      <div
        class="flex h-24 w-1/2 cursor-pointer items-center justify-center rounded-lg border border-gray-200 bg-gray-200/60 text-gray-900 shadow-xl transition-colors duration-200 ease-out hover:bg-gradient-to-r hover:from-blue-800 hover:from-20% hover:to-blue-500 hover:text-white active:scale-99"
        @click="isEditing ? toggleEditor() : startEditing()"
      >
        {{
          showEditor ? 'Hide editor' : isEditing ? 'Edit Bylaws' : 'Edit Bylaws'
        }}
        <UIcon
          :name="
            showEditor
              ? 'mdi:eye-off'
              : isEditing
                ? 'mdi:application-edit'
                : 'mdi:book-open-page-variant'
          "
          size="22"
          class="ml-1"
        />
      </div>
    </div>

    <div
      v-if="loggedIn && showEditor"
      class="editor-container mx-auto mt-6 w-[70%] rounded border p-4 shadow"
    >
      <TiptapEditor ref="editorRef" v-model="content" :show-image="false" />
      <div
        class="mt-4 flex flex-col items-stretch justify-between gap-4 sm:flex-row"
      >
        <div class="flex w-full flex-col gap-3 sm:w-auto sm:flex-row">
          <button
            class="flex w-full cursor-pointer items-center justify-center rounded-sm border border-gray-200 bg-gray-200/60 px-7 py-7 text-xl text-gray-900 shadow-xl transition-colors duration-200 ease-out hover:bg-gradient-to-r hover:from-green-800 hover:from-20% hover:to-green-500 hover:text-white active:scale-99 sm:w-auto"
            @click="saveChanges"
          >
            Save changes
            <UIcon name="mdi:upload" size="22" class="ml-1" />
          </button>

          <button
            class="flex w-full cursor-pointer items-center justify-center rounded-sm border border-gray-200 bg-gray-200/60 px-7 py-7 text-xl text-gray-900 shadow-xl transition-colors duration-200 ease-out hover:bg-gradient-to-r hover:from-blue-800 hover:from-20% hover:to-blue-500 hover:text-white active:scale-99 sm:w-auto"
            @click="cancelEditing"
          >
            Cancel
          </button>
        </div>
      </div>
    </div>
    <div
      class="relative bottom-5 mx-auto mt-12 w-[70%] rounded-lg bg-gray-50 p-4 shadow-lg"
    >
      <div v-html="bylawsHtml" />
    </div>
  </div>
</template>

<style scoped>
::v-deep img {
  max-width: 100%;
  height: auto;
  max-height: 100%;
  object-fit: contain;
  margin: auto;
  border: 1px solid #e5e7eb;
  border-radius: 12px;
  box-shadow: 0 10px 25px rgba(0, 0, 0, 0.1);
  transition:
    transform 0.3s ease,
    box-shadow 0.3s ease;
  display: block;
}

::v-deep table {
  width: 100%;
  border-collapse: collapse;
  object-fit: contain;
  max-width: 100%;
  min-width: 100%;
  table-layout: auto;
  overflow-x: auto;
}

::v-deep th,
::v-deep td {
  border: 1px solid #000000;
  padding: 0.5rem;
  word-break: break-word;
  overflow-wrap: break-word;
  white-space: normal;
  vertical-align: top;
  box-sizing: border-box;
}

::v-deep thead {
  background-color: #f3f4f6;
  font-weight: 600;
}

::v-deep tbody {
  max-width: 100%;
}
</style>
