<template>
  <div>
    <div class="flex items-center justify-center mb-6">
      <div
        class="flex h-24 w-1/2 cursor-pointer items-center justify-center
               rounded-lg border border-gray-200 bg-gray-200/60 text-gray-900 shadow-xl
               transition-colors duration-200 ease-out
               hover:bg-gradient-to-r hover:from-blue-800 hover:from-20% hover:to-blue-500
               hover:text-white active:scale-99"
        @click="toggleEditor"
      >
        {{
          showEditor
            ? 'Hide editor'
            : isEditing
              ? 'Edit article'
              : 'Open editor'
        }}
        <UIcon
          :name="
            showEditor
              ? 'mdi:eye-off'
              : isEditing
                ? 'mdi:book-edit'
                : 'mdi:book-open-page-variant'
          "
          size="22"
          class="ml-1"
        />
      </div>
    </div>
    <div
      v-show="showEditor"
      class="editor-container mx-auto mb-6 w-[80%] rounded border p-4 shadow"
    >
      <input
        v-model="title"
        placeholder="Tytuł bloga"
        class="mb-4 w-full rounded border px-2 py-1"
      >
      <!--Edytor-->
      <TiptapEditor ref="editorRef" v-model="content" :show-image="true" />
      <div class="mt-4 flex flex-col sm:flex-row justify-between items-stretch gap-4">
        <div class="flex flex-col sm:flex-row gap-3 w-full sm:w-auto">
          <button
            class="flex w-full sm:w-auto text-xl px-7 py-7 cursor-pointer items-center justify-center
                   rounded-sm border border-gray-200 bg-gray-200/60 text-gray-900 shadow-xl
                   transition-colors duration-200 ease-out
                   hover:bg-gradient-to-r hover:from-green-800 hover:from-20% hover:to-green-500
                   hover:text-white active:scale-99"
            @click="saveArticle"
          >
            {{ isEditing ? 'Save changes' : 'Send article' }}
            <UIcon name="mdi:book-plus" size="22" class="ml-1" />
          </button>
          <button
            class="flex w-full sm:w-auto text-xl px-7 py-7 cursor-pointer items-center justify-center
                   rounded-sm border border-gray-200 bg-gray-200/60 text-gray-900 shadow-xl
                   transition-colors duration-200 ease-out
                   hover:bg-gradient-to-r hover:from-blue-800 hover:from-20% hover:to-blue-500
                   hover:text-white active:scale-99"
            @click="cancelEdit"
          >
            Cancel
          </button>
        </div>
        <div v-if="isEditing" class="w-full sm:w-auto mt-3 sm:mt-0">
          <button
            class="flex w-full sm:w-auto text-xl px-7 py-7 cursor-pointer items-center justify-center
                   rounded-sm border border-gray-200 bg-gray-200/60 text-gray-900 shadow-xl
                   transition-colors duration-200 ease-out
                   hover:bg-gradient-to-r hover:from-red-800 hover:from-20% hover:to-red-500
                   hover:text-white active:scale-99"
            @click="deleteArticleById(editingId!)"
          >
            Delete article
            <UIcon name="mdi:book-remove" size="22" class="ml-1" />
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import type { EditorData, Article } from '~/types'
//props
const props = defineProps<{
  article?: Article
}>()
//emits
const emit = defineEmits<{
  (e: 'submitted' | 'cancel'): void
}>()
//refs
const title = ref<string>('')
const showEditor = ref<boolean>(false)
const isEditing = ref<boolean>(false)
const editingId = ref<number | null>()
const API_URL = useRuntimeConfig().public.API_URL
const toast = useToast()
const content = ref<EditorData>({
  html: '',
  images: [],
})
const editorRef = ref()
//log state
const loggedIn = useState('loggedIn')
//watches
watch(
  () => props.article,
  (newArticle) => {
    if (newArticle) {
      title.value = newArticle.title
      content.value = {
        html: newArticle.content,
        images: [...newArticle.content || []],
      }
      editingId.value = newArticle.id
      isEditing.value = true
      showEditor.value = true
    } else {
      resetForm()
    }
  },
  { immediate: true },
)

//toggle editor
function toggleEditor() {
  showEditor.value = !showEditor.value
  if (!showEditor.value) resetForm()
}
//save article
async function saveArticle() {
  if (!loggedIn) {
    alert('You must be logged in to save article.')
    return
  }

  if (!title.value.trim()) {
    toast.add({
        title: 'Warning',
        description: 'Title must be fullfilled to save article',
        color: 'error'
      })
    return
  }
  //send content
  const payload = {
    title: title.value,
    content: content.value.html,
    photoFileNames: [...content.value.images || []],
  }

  try {
    if (isEditing.value && editingId.value !== null) {
      //update article
      await $fetch(`${API_URL}/api/bulletin/${editingId.value}`, {
        method: 'PUT',
        body: payload,
        credentials: 'include',
      })

      toast.add({
        title: 'Success',
        description: 'Successfully edited',
        color: 'success',
      })
    } else {
      //post article
      await $fetch(`${API_URL}/api/bulletin`, {
        method: 'POST',
        body: payload,
        credentials: 'include',
      })
      toast.add({
        title: 'Success',
        description: 'Article has been successfully uploaded',
        color: 'success',
      })
    }

    emit('submitted')
    resetForm()
    showEditor.value = false
  } catch {
    toast.add({
      title: 'Error',
      description: 'Something went wrong pleas try again',
      color: 'error',
    })
  }
}
//editing function
function startEditing(article: Article) {
  title.value = article.title
  content.value = {
    html:
      typeof article.content === 'string'
        ? article.content
        : article.content,
    images: [...(article.photos ?? [])],
  }
  editingId.value = article.id
  isEditing.value = true
  showEditor.value = true
}
//exposed functions
defineExpose({
  startEditing,
  deleteArticleById,
})
//delete article
async function deleteArticleById(id: number) {
  console.log(id)
  try {
    await $fetch(`${API_URL}/api/bulletin/${id}`, {
      method: 'DELETE',
      credentials: 'include',
    })

    toast.add({
      title: 'Success',
      description: 'Article has been delated',
      color: 'success',
    })

    if (editingId.value === id || editingId.value === null) resetForm()
    showEditor.value = false
    emit('submitted')
  } catch {
    toast.add({
      title: 'Fail',
      description: 'Could not delete article. Pleas try again.',
      color: 'error',
    })
  }
}
//cancel edit
function cancelEdit() {
  emit('cancel')
  resetForm()
  showEditor.value = false
}
//reset editor value
function resetForm() {
  title.value = ''
  content.value = { html: '', images: [] }
  editingId.value = null
  isEditing.value = false
}
</script>

<style scoped>
</style>
