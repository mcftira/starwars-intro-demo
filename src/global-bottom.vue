<script setup>
/* Perzisztens Star Wars csillagmező + ködfoltok — az EGÉSZ deck alatt,
   a dia-váltásokon is átível (Slidev global layer, nem mountolódik újra). */
import { onMounted, onBeforeUnmount } from 'vue'

let cv, cx, stars = [], raf = 0, onResize = null

onMounted(() => {
  cv = document.getElementById('gbg-stars')
  if (!cv) return
  cx = cv.getContext('2d')
  const seed = () => {
    cv.width = innerWidth; cv.height = innerHeight
    stars = Array.from({ length: 340 }, () => ({
      x: Math.random()*cv.width, y: Math.random()*cv.height,
      r: Math.random()*1.35+.2, p: Math.random()*Math.PI*2, s: .006+Math.random()*.018
    }))
  }
  seed()
  onResize = () => seed()
  addEventListener('resize', onResize)
  const draw = () => {
    cx.clearRect(0,0,cv.width,cv.height)
    for (const s of stars){
      s.p += s.s
      cx.globalAlpha = .25 + Math.abs(Math.sin(s.p))*.6
      cx.fillStyle = '#fff'
      cx.fillRect(s.x, s.y, s.r, s.r)
    }
    cx.globalAlpha = 1
    raf = requestAnimationFrame(draw)
  }
  draw()
})

onBeforeUnmount(() => { cancelAnimationFrame(raf); onResize && removeEventListener('resize', onResize) })
</script>

<template>
  <div class="gbg-root">
    <canvas id="gbg-stars"></canvas>
    <div class="gbg-nebula n1"></div>
    <div class="gbg-nebula n2"></div>
  </div>
</template>

<style scoped>
.gbg-root{position:absolute;inset:0;background:var(--lp-bg);overflow:hidden;pointer-events:none}
#gbg-stars{position:absolute;inset:0}
.gbg-nebula{position:absolute;border-radius:50%;filter:blur(110px);opacity:.14}
.gbg-nebula.n1{width:55vw;height:55vw;left:-18vw;top:-22vw;background:radial-gradient(circle,#23355e,transparent 65%)}
.gbg-nebula.n2{width:50vw;height:50vw;right:-16vw;bottom:-20vw;background:radial-gradient(circle,#0d3a44,transparent 65%)}
</style>
