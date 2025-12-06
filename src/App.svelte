<script lang="ts">
import { BDFFont } from 'bdf-canvas'
import b16Bdf from './assets/b16.bdf?raw'

let canvas = $state<HTMLCanvasElement | null>(null)
let text = $state('')
let data = $state('')

$effect(() => {
  if (!text) return
  if (!canvas) return
  
  const ctx = canvas.getContext('2d')
  if (!ctx) return
  ctx.reset()

  const font = new BDFFont(b16Bdf)
  canvas.width = ((font.measureText(text).width-8>>4)+1)*16
  canvas.height = 16
  ctx.fillStyle = '#fff'
  font.drawText(ctx, text, 0, 13)
  const pixelData = ctx.getImageData(0, 0, canvas.width, canvas.height).data

  let tmp = ''
  for (let y = 0; y < 16; y++) {
    for (let x = 0; x < canvas.width/16; x++) {
      let bits = 0
      for (let b = 0; b < 16; b++) {
        bits <<= 1
        if (pixelData[(y * canvas.width + 16 * x + b) * 4]) bits |= 1
      }
      tmp += `0x${bits.toString(16)},`
    }
  }
  data = `uint16_t data[] = {${tmp.slice(0, -1)}};`
})
</script>

<main class="container mx-auto p-4">
  <h1 class="text-4xl font-bold uppercase mb-4">text2bitmap</h1>
  <p>16x16ビットマップフォントのテキストをuint16_t配列にするツール</p>
  <div class="mt-8">
    <input bind:value={text} class="input" />
    {#if text}
      <div class="w-full overflow-auto mb-4">
        <canvas bind:this={canvas} height="16" class="bg-black mt-8 h-8" style="image-rendering: pixelated;"></canvas>
      </div>
      <textarea readonly class="w-full field-sizing-content min-h-50">{data}</textarea>
    {/if}
  </div>
</main>
