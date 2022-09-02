<script setup lang="ts">
const onFileChange = async (e: Event) => {
  const target = e.target as HTMLInputElement
  if (!target.files || !target.files.length) {
    return
  }
  const file = target.files[0]

  const url = URL.createObjectURL(file)

  const task = await pdfjsLib.getDocument(url)

  const pdf = await task.promise

  await renderPdf(pdf)
}

const renderPdf = async (pdf: any) => {
  console.error(pdf)
  const page = await pdf.getPage(1);
  const viewport = page.getViewport({scale: 1});
  console.error(page, viewport)

  const canvas = <HTMLCanvasElement>document.getElementById("canvas")
  const context = canvas.getContext("2d");
  canvas.height = viewport.height;
  canvas.width = viewport.width;


  const renderContext = { canvasContext: context, viewport };
  await page.render(renderContext);
}
</script>

<template>
  <div>
    <input type="file" @change="onFileChange">
  </div>

  <div id="canvas_container">
    <canvas id="canvas"></canvas>
  </div>
</template>

<style scoped>
#canvas {
  width: 500px;
}
</style>
