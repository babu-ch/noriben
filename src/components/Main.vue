<script setup lang="ts">
import {onMounted, ref} from 'vue'

const isFileSelected = ref(false)
const generatedPngDataURL = ref("") // 最終的に出力するpng
const isCapture = ref(false) // pngに変換するときのflg
const pdfImage = ref(new Image()) // 読み込んだpdfを画像にしたやつ
const canvas = ref(<HTMLCanvasElement>{})
const mouseStatus = {
  start: {x: 0, y: 0},
  current: {x: 0, y: 0},
  isDragging: false
}
// 矩形描画用
interface Rect {
  startX: number;
  startY: number;
  endX: number;
  endY: number;
}
const drawRects: Rect[] = [];

onMounted(() => {
  anim();
})

/**
 * 描画処理main
 */
async function anim() {
  const canvasContext = canvas.value.getContext("2d");
  if (!canvasContext) {
    return
  }
  canvasContext.fillStyle = "rgba(0,0,0,0)";
  canvasContext.fillRect(0, 0, canvas.value.width, canvas.value.height);

  if (pdfImage.value.src) {
    canvasContext.drawImage(pdfImage.value, 0, 0)
  }

  canvasContext.fillStyle = "rgb(0, 0, 0)";
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
  // クリックハンドラでtoDataURLをするとどのタイミングでtoDataURLされるかわからない
  // 不完全な画像になるため描画処理の最後でoutputする
  if (isCapture.value) {
    isCapture.value = false
    generatedPngDataURL.value = canvas.value.toDataURL("image/png")
  }
  requestAnimationFrame(anim);
}

/**
 * ファイル読み込み
 * @param e
 */
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

  await setPdfImage(pdf)
}

/**
 * pdfをcanvasにrenderして画像として持っておく
 * 毎frame renderしたりすると重いファイルでチラついたので
 * @param pdf
 */
const setPdfImage = async (pdf: any) => {
  const canvasContext = canvas.value.getContext("2d");
  const page = await pdf.getPage(1);
  const viewport = page.getViewport({scale: canvas.value.width / page.getViewport({scale: 1}).width})

  canvas.value.height = viewport.height;

  const renderContext = { canvasContext: canvasContext, viewport }

  await page.render(renderContext).promise

  pdfImage.value.src = canvas.value.toDataURL("image/png")
}

/**
 * いろいろreset
 */
const reset = () => {
  generatedPngDataURL.value = ""
  isFileSelected.value = false
  drawRects.splice(0)
  pdfImage.value.src = ""
}

/**
 * mouseStatusの更新
 * @param e
 */
const mousedown = (e: MouseEvent) => {
  mouseStatus.isDragging = true
  mouseStatus.start.x = e.offsetX
  mouseStatus.start.y = e.offsetY
  mouseStatus.current.x = e.offsetX
  mouseStatus.current.y = e.offsetY
}
/**
 * mouseStatusの更新
 * drawRectsにpush
 * @param e
 */
const mouseup = (e: MouseEvent) => {
  mouseStatus.isDragging = false
  drawRects.push(<Rect>{
    startX: mouseStatus.start.x,
    startY: mouseStatus.start.y,
    endX: e.offsetX,
    endY: e.offsetY,
  })
}
/**
 * mouseStatusの更新
 * @param e
 */
const mousemove = (e: MouseEvent) => {
  mouseStatus.current.x = e.offsetX
  mouseStatus.current.y = e.offsetY
}
</script>

<template>
  <div v-if="!isFileSelected">
    <input type="file" @change="onFileChange" accept=".pdf">
  </div>
  <div v-else>
    <button @click="reset">reset</button>
    <button @click="isCapture = true">to png</button>
  </div>

  <div v-if="generatedPngDataURL.length">
    <p>↓PNG↓</p>
    <img :src="generatedPngDataURL" alt="">
  </div>
  <canvas id="canvas" ref="canvas" v-show="!generatedPngDataURL.length && isFileSelected"
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
