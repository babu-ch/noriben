<script setup lang="ts">
import {computed, onMounted, ref, watch} from 'vue'

const isFileSelected = ref(false)
const canvas = ref(<HTMLCanvasElement>{})
const mouseStatus = {
  start: {x: 0, y: 0},
  end: {x: 0, y: 0},
  current: {x: 0, y: 0},
  isDragging: false
}
interface Rect {
  startX: number;
  startY: number;
  endX: number;
  endY: number;
}
const drawRects: Rect[] = [];

const pdfStatus = {
  page: <any>null,
  renderContext: <any>null
}

onMounted(() => {
  anim();
})
async function anim() {
  const canvasContext = canvas.value.getContext("2d");
  if (!canvasContext) {
    return
  }

  canvasContext.fillStyle = "rgba(0,0,0,0)";
  canvasContext.fillRect(0, 0, canvas.value.width, canvas.value.height);

  if (pdfStatus.page) {
    await pdfStatus.page.render(pdfStatus.renderContext).promise
  }

  canvasContext.fillStyle = "rgb(0, 0, 255)";
  if (mouseStatus.isDragging) {
    const startX = mouseStatus.start.x
    const startY = mouseStatus.start.y
    const currentX = mouseStatus.current.x
    const currentY = mouseStatus.current.y
    canvasContext.fillRect( startX, startY, currentX - startX, currentY - startY )
  }

  drawRects.forEach((rect) => {
    canvasContext.fillRect( rect.startX, rect.startY, rect.endX - rect.startX, rect.endY - rect.startY)
  })
  requestAnimationFrame(anim)
}

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

  await setPdfStatus(pdf)
}

const setPdfStatus = async (pdf: any) => {
  const canvasContext = canvas.value.getContext("2d");
  const page = await pdf.getPage(1);
  const viewport = page.getViewport({scale: canvas.value.width / page.getViewport({scale: 1}).width})

  canvas.value.height = viewport.height;

  const renderContext = { canvasContext: canvasContext, viewport }

  pdfStatus.page = page
  pdfStatus.renderContext = renderContext
}

const clear = () => {
  const canvasContext = canvas.value.getContext("2d");
  if (!canvasContext) {
    return
  }
  isFileSelected.value = false
  canvasContext.clearRect(0, 0, canvas.value.width, canvas.value.height)
}

const download = () => {
  console.log('ok')
}

const mousedown = (e: MouseEvent) => {
  mouseStatus.isDragging = true
  mouseStatus.start.x = e.offsetX
  mouseStatus.start.y = e.offsetY
  mouseStatus.current.x = e.offsetX
  mouseStatus.current.y = e.offsetY
}
const mouseup = (e: MouseEvent) => {
  mouseStatus.end.x = e.offsetX
  mouseStatus.end.y = e.offsetY
  mouseStatus.isDragging = false
  drawRects.push(<Rect>{
    startX: mouseStatus.start.x,
    startY: mouseStatus.start.y,
    endX: mouseStatus.end.x,
    endY: mouseStatus.end.y,
  })
}
const mousemove = (e: MouseEvent) => {
  mouseStatus.current.x = e.offsetX
  mouseStatus.current.y = e.offsetY
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

  <canvas id="canvas" ref="canvas"
          @mousedown="mousedown"
          @mouseup="mouseup"
          @mousemove="mousemove"
          width="800"
  ></canvas>
</template>

<style scoped>
#canvas {
  width: 800px;
  background: gray;
}
</style>
