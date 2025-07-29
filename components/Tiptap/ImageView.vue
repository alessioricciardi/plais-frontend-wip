<template>
  <NodeViewWrapper
    :key="align"
    :class="[
      'group block w-fit max-w-full relative border border-gray-300 rounded shadow bg-white align-center',
    ]"
  >
    <img
      ref="imgEl"
      :src="node.attrs.src"
      :alt="node.attrs.alt || ''"
      :title="node.attrs.title || ''"
      :width="node.attrs.width"
      :height="node.attrs.height"
      class="block rounded pointer-events-auto select-none"
      draggable="false"
      @dragstart.prevent
    >

    <!-- Resize handle -->
    <div
      class="absolute bottom-0 right-0 w-3 h-3 bg-gray-600 rounded cursor-nwse-resize hidden group-hover:block z-10"
      @mousedown.prevent="startResize"
    />

    <!-- Drag handle -->
    <div
      class="drag-handle absolute -top-3 left-1/2 -translate-x-1/2 w-5 h-5 bg-blue-500 rounded-full cursor-grab z-10 flex items-center justify-center text-white text-xs"
      title="Przeciągnij"
      draggable="true"
      data-drag-handle
    >
      ☰
    </div>
  </NodeViewWrapper>
</template>

<script setup lang="ts">
import { NodeViewWrapper, type NodeViewProps } from '@tiptap/vue-3'

const props = defineProps<NodeViewProps>()
const { node, updateAttributes } = props

const imgEl = ref<HTMLImageElement | null>(null)
const align = ref(node.attrs.dataAlign || 'left')

// Uaktualnia align reactive, kiedy node.attrs.dataAlign się zmienia
watch(
  () => node.attrs.dataAlign,
  (newVal) => {
    align.value = newVal || 'left'
  }
)

let resizeStartX = 0
let resizeStartY = 0
let startWidth = 0
let startHeight = 0

function onMouseMoveResize(e: MouseEvent) {
  const newWidth = Math.max(50, startWidth + (e.clientX - resizeStartX))
  const newHeight = Math.max(50, startHeight + (e.clientY - resizeStartY))

  updateAttributes({
    width: `${newWidth}px`,
    height: `${newHeight}px`,
  })

  if (imgEl.value) {
    imgEl.value.width = newWidth
    imgEl.value.height = newHeight
  }
}

function onMouseUpResize() {
  document.removeEventListener('mousemove', onMouseMoveResize)
  document.removeEventListener('mouseup', onMouseUpResize)
}

function startResize(e: MouseEvent) {
  if (!imgEl.value) return

  resizeStartX = e.clientX
  resizeStartY = e.clientY
  startWidth = imgEl.value.offsetWidth
  startHeight = imgEl.value.offsetHeight

  document.addEventListener('mousemove', onMouseMoveResize)
  document.addEventListener('mouseup', onMouseUpResize)
}

</script>



<style scoped>
.align-left {
  margin-left: 0;
  margin-right: auto;
}
.align-center {
  margin-left: auto;
  margin-right: auto;
}
.align-right {
  margin-left: auto;
  margin-right: 0;
}

</style>