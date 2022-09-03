<script setup lang="ts">
import {computed, ref} from 'vue'

const isFileSelected = ref(false)
const canvas = ref(<HTMLCanvasElement>{})
const canvasContext = computed(() => {
    return canvas.value.getContext("2d")
})
const mouseStatus = ref({
  start: {x: 0, y: 0},
  end: {x: 0, y: 0},
  current: {x: 0, y: 0},
})

const onFileChange = async (e: Event) => {
  const target = e.target as HTMLInputElement
  if (!target.files || !target.files.length) {
    return
  }
  isFileSelected.value = true

  const file = target.files[0]

  const url = URL.createObjectURL(file)

  const task = await pdfjsLib.getDocument(url)

  const pdf = await task.promise

  await renderPdf(pdf)
}

const renderPdf = async (pdf: any) => {
  const page = await pdf.getPage(1);
  const viewport = page.getViewport({scale: 1});

  canvas.value.height = viewport.height;
  canvas.value.width = viewport.width;

  const renderContext = { canvasContext: canvasContext.value, viewport };
  await page.render(renderContext);
}

const clear = () => {
  isFileSelected.value = false
  canvasContext.value?.clearRect(0, 0, canvas.value.width, canvas.value.height)
}

const download = () => {
  console.log('ok')
}

const mousedown = (e: MouseEvent) => {
  mouseStatus.value.start.x = e.offsetX
  mouseStatus.value.start.y = e.offsetY
}
const mouseup = (e: MouseEvent) => {
  mouseStatus.value.end.x = e.offsetX
  mouseStatus.value.end.y = e.offsetY
}
const mousemove = (e: MouseEvent) => {
  mouseStatus.value.current.x = e.offsetX
  mouseStatus.value.current.y = e.offsetY
}
</script>

<template>
  <div v-if="!isFileSelected">
    <input type="file" @change="onFileChange">
  </div>
  <div v-else>
    <button @click="clear">clear</button>
    <button @click="download">download</button>
  </div>

  <div id="canvas_container">
    <canvas id="canvas" ref="canvas"
            @mousedown="mousedown"
            @mouseup="mouseup"
            @mousemove="mousemove"
    ></canvas>
  </div>
</template>

<style scoped>
#canvas {
  width: 500px;
}
</style>
