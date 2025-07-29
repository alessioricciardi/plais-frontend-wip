<script setup lang="ts">
// Editor core
import { EditorContent, useEditor, VueNodeViewRenderer } from '@tiptap/vue-3'
import StarterKit from '@tiptap/starter-kit'
import type {Editor} from '@tiptap/vue-3'
// Extensions
import TaskList from '@tiptap/extension-task-list'
import TaskItem from '@tiptap/extension-task-item'
import Underline from '@tiptap/extension-underline'
import TextAlign from '@tiptap/extension-text-align'
import SuperScript from '@tiptap/extension-superscript'
import Subscript from '@tiptap/extension-subscript'
import Highlight from '@tiptap/extension-highlight'
import Image from '@tiptap/extension-image'
import Dropcursor from '@tiptap/extension-dropcursor'
import Gapcursor from '@tiptap/extension-gapcursor'
import Link from '@tiptap/extension-link'
import Table from '@tiptap/extension-table'
import TableCell from '@tiptap/extension-table-cell'
import TableHeader from '@tiptap/extension-table-header'
import TableRow from '@tiptap/extension-table-row'
import type { DropdownMenuItem } from '@nuxt/ui'
import ImageView from './ImageView.vue'
import { TextSelection } from 'prosemirror-state'


// Props & Emits
interface EditorData {
  html: string
  images?: string[]
}

const props = defineProps<{ modelValue: EditorData; showImage?: boolean }>()
const emit = defineEmits<{
  (e: 'update:modelValue', value: EditorData): void
}>()

const fileInput = ref<HTMLInputElement | null>(null)

const API_URL = useRuntimeConfig().public.API_URL
const toast = useToast()


const CustomImage = Image.extend({
  name: 'image',
  group: 'block',
  draggable: true,
  selectable: true,

  addAttributes() {
    return {
      src: { default: null },
      alt: { default: null },
      title: { default: null },
      width: {
        default: 'auto',
        parseHTML: (el) => el.getAttribute('width') || 'auto',
        renderHTML: (attrs) => ({ width: attrs.width }),
      },
      height: {
        default: 'auto',
        parseHTML: (el) => el.getAttribute('height') || 'auto',
        renderHTML: (attrs) => ({ height: attrs.height }),
      },
      dataAlign: {
        default: 'left',
        parseHTML: (el) => el.getAttribute('data-align') || 'left',
        renderHTML: (attrs) => ({
        'data-align': attrs.dataAlign || 'left',
        class: `align-${attrs.dataAlign || 'left'}`,
        }),
      },
    }
  },

  renderHTML({ node, HTMLAttributes }) {
    return [
      'img',
      {
        ...HTMLAttributes,
        src: node.attrs.src,
        alt: node.attrs.alt,
        title: node.attrs.title,
        width: node.attrs.width,
        height: node.attrs.height,
        'data-align': node.attrs.dataAlign,
        class: `align-${node.attrs.dataAlign}`,
      },
    ]
  },

  addNodeView() {
    return VueNodeViewRenderer(ImageView)
  },
})

const editor  = useEditor({
  content: props.modelValue.html,
  extensions: [
    StarterKit,
    Underline,
    Highlight,
    TextAlign.configure({ types: ['heading', 'paragraph'] }),
    TaskList,
    TaskItem.configure({ nested: true }),
    SuperScript,
    Subscript,
    CustomImage,
    Dropcursor,
    Gapcursor,
    Link.configure({
      defaultProtocol: 'https',
      protocols: ['https'],
      HTMLAttributes: { target: '_blank',
        class: 'text-blue-600 underline font-medium hover:text-blue-800',
       },
    }),
    Table.configure({ resizable: true}),
    TableCell,
    TableHeader,
    TableRow,
  ],
  onUpdate: ({ editor }) => {
    if (!editor) return
    emit('update:modelValue', {
      html: editor.getHTML(),
      images: getCurrentImagesFromDocument(editor),
    })
  },
})

watch(
  () => props.modelValue.html,
  (newHtml) => {
    if (!editor.value) return

    const currentHtml = editor.value.getHTML().trim()
    if (newHtml.trim() !== currentHtml) {
      editor.value.commands.setContent(newHtml, false)
      nextTick(() => {
        return emit('update:modelValue', {
          html: editor.value?.getHTML() || '',
          images: getCurrentImagesFromDocument(editor.value),
        })
      })
    }
  },
)


function getCurrentImagesFromDocument(editorInstance?: Editor): string[] {
  if (!editorInstance) return []

  const imageUrls: string[] = []
  editorInstance.state.doc.descendants((node) => {
    if (node.type.name === 'image' && node.attrs.src) {
      const src: string = node.attrs.src
      const fileName = src.split('/').pop()
      if (fileName) imageUrls.push(fileName)
    }
  })

  return imageUrls
}


const onImageUpload = async (event: Event) => {
  const target = event.target as HTMLInputElement
  const file = target.files?.[0]
  if (!file || !editor.value) return

  const formData = new FormData()
  formData.append('file', file)

  try {
    const response = await $fetch<{ fileName: string; url: string }>(
      `${API_URL}/api/Image/bulletin`,
      {
        method: 'POST',
        body: formData,
        credentials: 'include',
      },
    )

    const fullImageUrl = `${API_URL}${response.url}`

    editor.value.chain().focus().setImage({ src: fullImageUrl }).run()
    console.log(editor.value.getHTML())

    emit('update:modelValue', {
    html: editor.value.getHTML(),
    images: getCurrentImagesFromDocument(editor.value),
    })


    if (fileInput.value) fileInput.value.value = ''
  } catch {
    toast.add({
       title: 'Error',
      description: 'Failed to upload photo',
      color: 'error',
    })
  }
}

async function removeImage(fileName: string) {
  if (!editor.value) return

  const imageUrl = `${API_URL}/uploads/bulletins/${fileName}`

  const { state } = editor.value
  const tr = state.tr
  let updated = false

  state.doc.descendants((node, pos) => {
    if (node.type.name === 'image' && node.attrs.src === imageUrl) {
      tr.delete(pos, pos + node.nodeSize)
      updated = true
    }
  })

  if (updated) {
    editor.value.view.dispatch(tr)

    const newHtml = editor.value.getHTML()

    emit('update:modelValue', {
      html: newHtml,
      images: getCurrentImagesFromDocument(editor.value),
    })

    editor.value.commands.setContent(newHtml, false)
  }

  //Usuń z backendu
  try {
    await $fetch(`${API_URL}/api/Image/bulletin`, {
      method: 'DELETE',
      query: { fileName },
      credentials: 'include',
    })
  } catch (e) {
    console.error('Błąd usuwania obrazu z backendu:', e)
    alert('Nie udało się usunąć obrazu z serwera.')
  }
}
defineExpose({
  removeImage,
})

// Link modal
const showLinkModal = ref<boolean>(false)
const linkUrl = ref<string>('')
const linkText = ref<string>('')

function openLinkModal() {
  if (!editor.value) return

  const { from, to } = editor.value.state.selection
  const selectedText = editor.value.state.doc.textBetween(from, to)
  const linkMark = editor.value.getAttributes('link')

  linkUrl.value = linkMark?.href || ''
  linkText.value = selectedText || ''

  showLinkModal.value = true
}
function closeLinkModal() {
  showLinkModal.value = false
  linkUrl.value = ''
  linkText.value = ''
}

function applyLink() {
  if (!editor.value) return;

  const href = linkUrl.value.trim();
  const text = linkText.value.trim();

  if (href === '') {
    editor.value.chain().focus().unsetLink().run();
    closeLinkModal();
    return;
  }

  const chain = editor.value.chain().focus().extendMarkRange('link');

  if (editor.value.state.selection.empty) {
    // jeśli nie ma zaznaczenia – wstaw nowy tekst z linkiem
    chain.insertContent({
      type: 'text',
      text: text || href,
      marks: [{ type: 'link', attrs: { href } }],
    });
  } else {
    // jeśli jest zaznaczenie – po prostu nałóż link
    chain.setLink({ href });
  }

  chain.run();
  closeLinkModal();
}


// Keydown handler
const onKeydown = (event: KeyboardEvent) => {
  if (event.key !== ' ' || !editor.value) return;

  if (editor.value.isActive('link')) {
    event.preventDefault();

    editor.value
      .chain()
      .unsetMark('link')  // tylko dezaktywuj mark
      .insertContent(' ') // wstaw spację poza linkiem
      .focus()
      .run();
  }
};





onMounted(() => {
  editor.value?.view.dom?.addEventListener('keydown', onKeydown)
  const dom = editor.value?.view.dom
  if (!dom) return
  dom.addEventListener('dragstart', handleDragStart)
  dom.addEventListener('drop', handleDrop)
})

onBeforeUnmount(() => {
  editor.value?.view.dom?.removeEventListener('keydown', onKeydown)
  editor.value?.destroy()

  const dom = editor.value?.view.dom
  if (!dom) return

  dom.removeEventListener('dragstart', handleDragStart)
  dom.removeEventListener('drop', handleDrop)
})

function handleDragStart(event: DragEvent) {
  const view = editor.value?.view
  if (!view || !event.target) return

  const pos = view.posAtDOM(event.target as HTMLElement, 0)
  console.log('[DRAG START] Node at pos:', pos)
}

function handleDrop(event: DragEvent) {
  const view = editor.value?.view
  if (!view || !event.dataTransfer) return

  const pos = view.posAtCoords({ left: event.clientX, top: event.clientY })
  if (!pos) return

  console.log('[DROP] Drop position in doc:', pos.pos)

  const { state, dispatch } = view
  const tr = state.tr.setSelection(TextSelection.create(state.doc, pos.pos))
  dispatch(tr)
}

// Dropdown menu
const headerList = ref<DropdownMenuItem[]>([
  { icon: 'i-material-symbols-format-h1-rounded', slot: 'h1', label: '' },
  { icon: 'i-material-symbols-format-h2-rounded', slot: 'h2', label: '' },
  { icon: 'i-material-symbols-format-h3-rounded', slot: 'h3', label: '' },
  { icon: 'i-material-symbols-format-h4-rounded', slot: 'h4', label: '' },
  { icon: 'i-material-symbols-format-h5-rounded', slot: 'h5', label: '' },
  { icon: 'i-material-symbols-format-h6-rounded', slot: 'h6', label: '' },
])

const tableList = ref<DropdownMenuItem[]>([
  { icon: 'i-mdi-table-plus', slot: 'insert-table', label: '' },
  { icon: 'i-mdi-table-check', slot: 'fix-tables', label: '' },
  { icon: 'i-mdi-table-remove', slot: 'delete-table', label: '' },
  { icon: '', slot: 'add-column-before', label: '' },
  { icon: '', slot: 'add-column-after', label: '' },
  { icon: '', slot: 'delete-column', label: '' },
  { icon: '', slot: 'add-row-before', label: '' },
  { icon: '', slot: 'add-row-after', label: '' },
  { icon: '', slot: 'delete-row', label: '' },
  { icon: '', slot: 'toggle-header-row', label: '' },
  { icon: '', slot: 'toggle-header-column', label: '' },
  { icon: '', slot: 'toggle-header-cell', label: '' },
  { icon: '', slot: 'merge-cells', label: '' },
  { icon: '', slot: 'split-cell', label: '' },
])
</script>

<template>
  <div class="w-full gap-1 text-gray-900">
    <!-- Editor loaded -->
    <template v-if="editor">
      <!-- Formatters -->
      <div class="mb-4 flex flex-row flex-wrap gap-2">
        <!-- Bold -->
        <TiptapButton
          tooltip="Bold"
          :disabled-binding="!editor.can().chain().focus().toggleBold().run()"
          :class-binding="{
            'bg-gray-300/50': editor.isActive('bold'),
            'text-blue-600': editor.isActive('bold'),
            'cursor-pointer' : true
          }"
          icon="material-symbols:format-bold-rounded"
          @click.stop="editor.chain().focus().toggleBold().run()"
        />

        <!-- Italic -->
        <TiptapButton
          tooltip="Italic"
          :disabled-binding="!editor.can().chain().focus().toggleItalic().run()"
          
          :class-binding="{
            'bg-gray-300/50': editor.isActive('italic'),
            'text-blue-600': editor.isActive('italic'),
            'cursor-pointer' : true
          }"
          icon="material-symbols-format-italic-rounded"
          @click="editor.chain().focus().toggleItalic().run()"
        />

        <!-- Underline -->
        <TiptapButton
          tooltip="Underline"
          :class-binding="{
            'bg-gray-300/50': editor.isActive('underline'),
            'text-blue-600': editor.isActive('underline'),
            'cursor-pointer': true,
          }"
          icon="material-symbols:format-underlined-rounded"
          @click="editor.chain().focus().toggleUnderline().run()"
        />

        <!-- Strikethrough -->
        <TiptapButton
          tooltip="Strikethrough"
          :disabled-binding="!editor.can().chain().focus().toggleStrike().run()"
          :class-binding="{
            'bg-gray-300/50': editor.isActive('strike'),
            'text-blue-600': editor.isActive('strike'),
            'cursor-pointer': true,
          }"
          icon="i-material-symbols-format-strikethrough-rounded"
          @click="editor.chain().focus().toggleStrike().run()"
        />

        <!-- Highlight -->
        <TiptapButton
          tooltip="Highlight"
          :class-binding="{
            'bg-gray-300/50': editor.isActive('highlight'),
            'text-blue-600': editor.isActive('highlight'),
            'cursor-pointer': true,
          }"
          icon="i-material-symbols-highlighter-size-4"
          @click="editor.chain().focus().toggleHighlight().run()"
        />

        <!-- ALIGNMENT -->

        <!-- Align Left -->
        <TiptapButton
          tooltip="Text Align Left"
          icon="material-symbols:format-align-left-rounded"
          :class-binding="{
            'bg-gray-300/50': editor.isActive({ textAlign: 'left' }),
            'text-blue-600': editor.isActive({ textAlign: 'left' }),
            'cursor-pointer': true,
          }"
          @click="editor.chain().focus().setTextAlign('left').run()"
        />

        <!-- Align Center -->
        <TiptapButton
          tooltip="Text Align Center"
          icon="material-symbols:format-align-center-rounded"
          :class-binding="{
            'bg-gray-300/50': editor.isActive({ textAlign: 'center' }),
            'text-blue-600': editor.isActive({ textAlign: 'center' }),
            'cursor-pointer': true,
          }"
          @click="editor.chain().focus().setTextAlign('center').run()"
        />

        <!-- Align Right -->
        <TiptapButton
          tooltip="Text Align Right"
          icon="material-symbols:format-align-right"
          :class-binding="{
            'bg-gray-300/50': editor.isActive({ textAlign: 'right' }),
            'text-blue-600': editor.isActive({ textAlign: 'right' }),
            'cursor-pointer': true,
          }"
          @click="editor.chain().focus().setTextAlign('right').run()"
        />

        <!-- Superscript -->
        <TiptapButton
          tooltip="Superscript"
          :class-binding="{
            'bg-gray-300/50': editor.isActive('superscript'),
            'text-blue-600': editor.isActive('superscript'),
            'cursor-pointer': true,
          }"
          icon="i-material-symbols-superscript-rounded"
          @click="editor.chain().focus().toggleSuperscript().run()"
        />

        <!-- Subscript -->
        <TiptapButton
          tooltip="Subscript"
          :class-binding="{
            'bg-gray-300/50': editor.isActive('subscript'),
            'text-blue-600': editor.isActive('subscript'),
            'cursor-pointer': true,
          }"
          icon="i-material-symbols-subscript-rounded"
          @click="editor.chain().focus().toggleSubscript().run()"
        />

        <!-- Code -->
        <TiptapButton
          tooltip="Code"
          :disabled-binding="!editor.can().chain().focus().toggleCode().run()"
          class="cursor-pointer"
          :class-binding="{
            'bg-gray-300/50': editor.isActive('code'),
            'text-blue-600': editor.isActive('code'),
            'cursor-pointer': true,
          }"
          icon="i-material-symbols-code"
          @click="editor.chain().focus().toggleCode().run()"
        />

        <!-- Paragraph -->
        <TiptapButton
          tooltip="Paragraph"
          class="cursor-pointer"
          :class-binding="{
            'bg-gray-300/50': editor.isActive('paragraph'),
            'text-blue-600': editor.isActive('paragraph'),
            'cursor-pointer': true,
          }"
          icon="i-material-symbols-format-paragraph"
          @click="editor.chain().focus().setParagraph().run()"
        />
        <div v-if="showImage" class="flex items-center gap-2">
          <!--Images-->
          <input
            ref="fileInput"
            type="file"
            accept="image/*"
            class="hidden"
            @change="onImageUpload"
          >
          <!--Insert-->
          <TiptapButton
            tooltip="Insert Image"
            icon="i-material-symbols-image-rounded"
            :class-binding="{
            'cursor-pointer': true,
            }"
            @click="() => fileInput?.click()"
          />
          
        </div>
        <!-- Heading -->
        <UDropdownMenu
          :items="headerList"
          :popper="{ placement: 'bottom-start' }"
          :content="{
            align: 'center',
            side: 'right',
            sideOffset: 0,
            positionStrategy: 'absolute',
          }"
          :ui="{
            group: 'flex ',
            item: 'hover:bg-white',
          }"
        >
          <!-- Main Heading Button -->
          <UButton
            icon="i-material-symbols-h-mobiledata-rounded"
            variant="ghost"
            class="text-3xl text-gray-900 hover:bg-gray-300/80 cursor-pointer"
            :class="{
              'bg-gray-300/50': editor.isActive('heading'),
              'text-blue-600': editor.isActive('heading'),
            }"
          />

          <!-- Heading Dropdown -->
          <template #h1>
            <UButton
              icon="i-material-symbols-format-h1-rounded"
              variant="ghost"
              class="flex w-full text-2xl text-gray-900 cursor-pointer"
              :class="{
                'bg-gray-300/50': editor?.isActive('heading', {
                  level: 1,
                }),
                'text-blue-600': editor?.isActive('heading', {
                  level: 1,
                }),
              }"
              @click="editor?.chain().focus().toggleHeading({ level: 1 }).run()"
            />
          </template>

          <template #h2>
            <UButton
              icon="i-material-symbols-format-h2-rounded"
              variant="ghost"
              class="w-full text-2xl text-gray-900 hover:bg-gray-300/80 cursor-pointer"
              :class="{
                'bg-gray-300/50': editor?.isActive('heading', {
                  level: 2,
                }),
                'text-blue-600': editor?.isActive('heading', {
                  level: 2,
                }),
              }"
              @click="editor?.chain().focus().toggleHeading({ level: 2 }).run()"
            />
          </template>

          <template #h3>
            <UButton
              icon="i-material-symbols-format-h3-rounded"
              variant="ghost"
              class="w-full text-2xl text-gray-900 hover:bg-gray-300/80 cursor-pointer"
              :class="{
                'bg-gray-300/50': editor?.isActive('heading', {
                  level: 3,
                }),
                'text-blue-600': editor?.isActive('heading', {
                  level: 3,
                }),
              }"
              @click="editor?.chain().focus().toggleHeading({ level: 3 }).run()"
            />
          </template>

          <template #h4>
            <UButton
              icon="i-material-symbols-format-h4-rounded"
              variant="ghost"
              class="w-full text-2xl text-gray-900 hover:bg-gray-300/80 cursor-pointer"
              :class="{
                'bg-gray-300/50': editor?.isActive('heading', {
                  level: 4,
                }),
                'text-blue-600': editor?.isActive('heading', {
                  level: 4,
                }),
              }"
              @click="editor?.chain().focus().toggleHeading({ level: 4 }).run()"
            />
          </template>

          <template #h5>
            <UButton
              icon="i-material-symbols-format-h5-rounded"
              variant="ghost"
              class="w-full text-2xl text-gray-900 hover:bg-gray-300/80 cursor-pointer"
              :class="{
                'bg-gray-300/50': editor?.isActive('heading', {
                  level: 5,
                }),
                'text-blue-600': editor?.isActive('heading', {
                  level: 5,
                }),
              }"
              @click="editor?.chain().focus().toggleHeading({ level: 5 }).run()"
            />
          </template>

          <template #h6>
            <UButton
              icon="i-material-symbols-format-h6-rounded"
              variant="ghost"
              class="w-full text-2xl text-gray-900 hover:bg-gray-300/80 cursor-pointer"
              :class="{
                'bg-gray-300/50': editor?.isActive('heading', {
                  level: 6,
                }),
                'text-blue-600': editor?.isActive('heading', {
                  level: 6,
                }),
              }"
              @click="editor?.chain().focus().toggleHeading({ level: 6 }).run()"
            />
          </template>
        </UDropdownMenu>

        <!-- Lists -->

        <!-- Bullet List -->
        <TiptapButton
          tooltip="Bullet List"
          icon="i-material-symbols-format-list-bulleted-rounded"
          class="cursor-pointer"
          :class-binding="{
            'bg-gray-300/50': editor.isActive('bulletList'),
            'text-blue-600': editor.isActive('bulletList'),
            'cursor-pointer': true,
          }"
          @click="editor.chain().focus().toggleBulletList().run()"
        />

        <!-- Ordered List -->
        <TiptapButton
          tooltip="Ordered List"
          icon="i-material-symbols-format-list-numbered-rounded"
          class="cursor-pointer"
          :class-binding="{
            'bg-gray-300/50': editor.isActive('orderedList'),
            'text-blue-600': editor.isActive('orderedList'),
            'cursor-pointer': true,
          }"
          @click="editor.chain().focus().toggleOrderedList().run()"
        />

        <TiptapButton
          tooltip="Check List"
          icon="i-material-symbols-checklist-rounded"
          class="cursor-pointer"
          :class-binding="{
            'bg-gray-300/50': editor.isActive('taskList'),
            'text-blue-600': editor.isActive('taskList'),
            'cursor-pointer': true,
          }"
          @click="editor.chain().focus().toggleTaskList().run()"
        />

        <!--modal-->
        <!--test modal-->
        <div class="relative inline-block">
        <TiptapButton
          tooltip="Link"
          icon="i-material-symbols-attachment-rounded"
          :class-binding="{
            'bg-gray-200': showLinkModal,
            'text-blue-600': showLinkModal,
            'cursor-pointer': true,
          }"
          @click="openLinkModal"
        />
        
        <!-- Popup -->
        <div
          v-if="showLinkModal"
          ref="linkModal"
          class="absolute z-50 mt-2 w-80 rounded-xl bg-white p-6 shadow-xl"
        >
        <UButton
        icon="typcn:delete"
        class="cursor-pointer bg-red-500 text-lg text-gray-900 hover:bg-red-600 absolute top-3 right-3"
        @click="closeLinkModal"
        />
        
          <h2 class="mb-4 text-lg font-bold">Insert link</h2>
        
          <!-- Adres URL -->
          <label class="mb-2 block text-sm">URL:</label>
          <input
            v-model="linkUrl"
            type="text"
            class="mb-4 w-full rounded border px-3 py-2"
            placeholder="https://example.com"
          >
        
          <!-- Tekst linku -->
          <label class="mb-2 block text-sm">Text:</label>
          <input
            v-model="linkText"
            type="text"
            class="mb-4 w-full rounded border px-3 py-2"
            placeholder="click here"
          >
        
          <!-- Przyciski -->
          <div class="flex justify-end gap-2">
            <button
              class=" rounded bg-blue-600 px-4 py-2 text-white hover:bg-blue-700 cursor-pointer"
              @click="applyLink"
            >
              Insert
            </button>
          </div>
        </div>
      </div>

        
        <UDropdownMenu
          :items="tableList"
          :content="{
            align: 'start',
            side: 'right',
            sideOffset: 8,
            positionStrategy: 'absolute',
          }"
          :ui="{
            group: 'flex flex-col xl:flex-row max-w-xs xl:max-w-none',
            item: 'w-full xl:w-auto px-2 xl:px-4 text-left text-sm xl:text-base',
          }"
        >
          <!-- Main Heading Button -->
          <UButton
            tooltip="Table"
            icon="i-material-symbols-table-chart"
            variant="ghost"
            class="cursor-pointer text-3xl text-gray-900"
            :class="{
              'bg-gray-300/50': editor.isActive('table'),
              'text-blue-600': editor.isActive('table'),
            }"
          />

          <!-- Heading Dropdown -->
          <template #insert-table>
            <UButton
              icon="i-mdi-table-plus"
              variant="ghost"
              class="hover:bg-transparent cursor-pointer flex w-full text-3xl text-gray-900"
              title="Insert table"
              @click="
                editor.chain().focus().insertTable({ rows: 3, cols: 3 }).run()
              "
            />
          </template>

          <template #delete-table>
            <UButton
              :class-binding="{}"
              icon="i-mdi-table-remove"
              variant="ghost"
              class="hover:bg-transparent cursor-pointer flex w-full text-3xl text-gray-900"
              title="Delete table"
              @click="editor.chain().focus().deleteTable().run()"
              
              
            />
          </template>

          <template #fix-tables>
            <UButton
              
              icon="i-mdi-table-check"
              :class-binding="{}"
              variant="ghost"
              class="hover:bg-transparent cursor-pointer flex w-full text-3xl text-gray-900"
              title="Fix tables"
              @click="editor.chain().focus().fixTables().run()"
            />
          </template>

          <template #add-column-before>
            <UButton
             
              icon="material-symbols-light:add-column-left-rounded"
              :class-binding="{}"
              variant="ghost"
              class="hover:bg-transparent cursor-pointer flex w-full text-3xl text-gray-900"
              title="Add column before"
              @click="editor.chain().focus().addColumnBefore().run()"
            />
          </template>

          <template #add-column-after>
            <UButton
              
              icon="material-symbols-light:add-column-right"
              :class-binding="{}"
              variant="ghost"
              class="hover:bg-transparent cursor-pointer flex w-full text-3xl text-gray-900"
              title="Add column after"
              @click="editor.chain().focus().addColumnAfter().run()"
            />
          </template>

          <template #delete-column>
            <UButton
             
              icon="i-mdi-table-column-remove"
              :class-binding="{}"
              variant="ghost"
              class="hover:bg-transparent cursor-pointer flex w-full text-3xl text-gray-900"
              title="Delete column"
              @click="editor.chain().focus().deleteColumn().run()"
            />
          </template>
          <template #add-row-before>
            <UButton
             tooltip="Add Row Before"
              icon="i-material-symbols-add-row-above-rounded"
              :class-binding="{}"
              variant="ghost"
              class="hover:bg-transparent cursor-pointer flex w-full text-3xl text-gray-900"
              title="Add row before"
              @click="editor.chain().focus().addRowBefore().run()"
            />
          </template>
          <template #add-row-after>
            <UButton
              icon="i-material-symbols-add-row-below-rounded"
              :class-binding="{}"
              variant="ghost"
              class="hover:bg-transparent cursor-pointer flex w-full text-3xl text-gray-900"
              title="Add row after"
              @click="editor.chain().focus().addRowAfter().run()"
            />
          </template>
          <template #delete-row>
            <UButton
             
              icon="i-mdi-table-row-remove"
              :class-binding="{}"
              variant="ghost"
              class="hover:bg-transparent cursor-pointer flex w-full text-3xl text-gray-900"
              title="Delete row"
              @click="editor.chain().focus().deleteRow().run()"
            />
          </template>
          <template #toggle-header-row>
            <UButton
              icon="i-material-symbols-table-rows-rounded"
              :class-binding="{}"
              variant="ghost"
              class="hover:bg-transparent cursor-pointer flex w-full text-3xl text-gray-900"
              title="Toggle header row"
              @click="editor.chain().focus().toggleHeaderRow().run()"
            />
          </template>
          <template #toggle-header-column>
            <UButton
              
              icon="i-material-symbols-view-column-rounded"
              :class-binding="{}"
              variant="ghost"
              class="hover:bg-transparent cursor-pointer flex w-full text-3xl text-gray-900"
              title="Toggle header column"
              @click="editor.chain().focus().toggleHeaderColumn().run()"
            />
          </template>
          <template #toggle-header-cell>
            <UButton
              
              icon="i-material-symbols-square-rounded"
              :class-binding="{}"
              variant="ghost"
              class="hover:bg-transparent cursor-pointer flex w-full text-3xl text-gray-900"
              title="Toggle header cell"
              @click="editor.chain().focus().toggleHeaderCell().run()"
            />
          </template>
          <template #merge-cells>
            <UButton
              
              icon="i-mdi-table-merge-cells"
              :class-binding="{}"
              variant="ghost"
              class="hover:bg-transparent cursor-pointer flex w-full text-3xl text-gray-900"
              title="Merge cells"
              @click="editor.chain().focus().mergeCells().run()"
            />
          </template>
          <template #split-cell>
            <UButton
              icon="i-mdi-table-split-cell"
              :class-binding="{}"
              variant="ghost"
              class="hover:bg-transparent cursor-pointer flex w-full text-3xl text-gray-900"
              title="Split cells"
              @click="editor.chain().focus().splitCell().run()"
            />
          </template>
        </UDropdownMenu>
        <!--end-->
      </div>
      <!-- Text editor -->
      <EditorContent
        :editor="editor"
        class="tiptap min-h-30 w-full rounded-lg border-2 border-gray-900 p-2"
      />
      <div v-if="props.modelValue?.images?.length" class="mt-6">
        <h3 class="mb-2 text-sm font-bold">Send images:</h3>
        <ul class="flex list-none flex-wrap gap-4 p-0">
          <li
           v-for="(name, index) in props.modelValue.images"
          :key="index"
          class="relative"
          >
            <img
              :src="`${API_URL}/uploads/bulletins/${name}`"
              alt="Miniatura"
              class="h-24 w-24 rounded object-cover shadow"
            >
            <UButton
              icon="typcn:delete"
              class="absolute top-1 right-1 rounded-full bg-red-500 p-1 text-white hover:bg-red-600 cursor-pointer"
              @click="removeImage(name)"
            />
          </li>
        </ul>
      </div>
    </template>
    <!-- Editor loading -->
    <template v-else>
      <div class="flex justify-center">
        <span class="loading loading-spinner mr-3 text-blue-600" />
        Loading Editor...
      </div>
    </template>
  </div>
</template>

<style scoped>
img.align-center {
  display: block;
  margin-left: auto;
  margin-right: auto;
}


::v-deep(.tiptap) {
  max-width: 100%;
  width: 100%;
}
::v-deep(.tiptap img) {
  max-width: 100%;
  height: auto;
  object-fit: contain;
  user-select: none;
  pointer-events: auto;
  border-radius: 4px;
}

.drag-handle {
  cursor: grab;
  width: 16px;
  height: 16px;
  background-color: #3b82f6;
  color: white;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 12px;
  border-radius: 50%;
  box-shadow: 0 0 2px rgba(0,0,0,0.2);
  user-select: none;
  position: absolute;
  z-index: 10;
}

/* --- Wrapper dla resizable image --- */
.resizable-image-wrapper {
  display: inline-block;
  position: relative;
}

.resize-handle {
  position: absolute;
  bottom: 0;
  right: 0;
  width: 12px;
  height: 12px;
  background-color: #4b5563; /* szary */
  border-radius: 2px;
  cursor: nwse-resize;
  display: none;
  z-index: 20;
}

.resizable-image-wrapper:hover .resize-handle {
  display: block;
}

::v-deep(.tiptap table) {
    margin: 0;
    table-layout: fixed;
    width: 100%;
}

::v-deep(.tiptap table th),
::v-deep(.tiptap table td) {
  border: 1px solid #000000;
  padding: 0.5rem;
  word-break: break-word;
  white-space: normal;
  vertical-align: top;
  position: relative;
  box-sizing: border-box;
  overflow-wrap: break-word;
}

::v-deep(.tiptap thead) {
  background-color: #f0f0f0;
}

::v-deep(.column-resize-handle) {
  position: absolute;
  right: -2px;
  top: 0;
  bottom: 0;
  width: 6px;
  cursor: col-resize;
  z-index: 10;
  background-color: transparent;
}
::v-deep(.column-resize-handle:hover) {
  background-color: rgba(0, 0, 0, 0.15);
}

.image-container:active {
  cursor: grabbing;
}

::v-deep(.selectedCell)::after {
  content: "";
  position: absolute;
  background-color: rgba(59, 130, 246, 0.25); /* niebieski */
  top: 0;
  right: 0;
  bottom: 0;
  left: 0;
  pointer-events: none;
  z-index: 2;
}

::v-deep(.tableWrapper) {
    margin: 1.5rem 0;
    overflow-x: auto;
  }
</style>