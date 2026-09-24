<script setup>
/* Perzisztens Star Wars csillagmező + ködfoltok — az EGÉSZ deck alatt,
   a dia-váltásokon is átível (Slidev global layer, nem mountolódik újra).

   Teljesítmény: a csillagok lassan pislákolnak, ezért elég ~30 fps-en
   újrarajzolni őket; a fényerőt vödrökbe kvantáljuk, így keretenként
   néhány canvas-állapot-váltás van a korábbi 340 helyett; rejtett fülön
   a rajzolás teljesen leáll. A pislákolás sebessége időalapú, ezért
   a látvány független a képkocka-sebességtől. */
import { onMounted, onBeforeUnmount } from 'vue'

const STAR_COUNT = 340
const FRAME_MS = 1000 / 30     // a pislákolás ennél gyorsabban nem látszik
const BUCKETS = 8              // fényerő-lépcsők
const STEP_MS = 1000 / 60      // az eredeti fázissebesség referenciája

let cv, cx, stars = [], raf = 0, last = 0, running = false
let onResize = null, onVis = null, resizeTimer = 0
const buckets = Array.from({ length: BUCKETS }, () => [])

function seed() {
  cv.width = innerWidth; cv.height = innerHeight
  stars = Array.from({ length: STAR_COUNT }, () => ({
    x: Math.random()*cv.width, y: Math.random()*cv.height,
    r: Math.random()*1.35+.2, p: Math.random()*Math.PI*2, s: .006+Math.random()*.018
  }))
}

function start() {
  if (running || !cx) return
  running = true; last = 0
  raf = requestAnimationFrame(draw)
}
function stop() {
  running = false
  if (raf) { cancelAnimationFrame(raf); raf = 0 }
}

function draw(t) {
  if (!running) return
  raf = requestAnimationFrame(draw)
  if (!last) { last = t; return }
  const dt = t - last
  if (dt < FRAME_MS) return          // ezen a kereten nem rajzolunk
  last = t
  const k = Math.min(dt, 100) / STEP_MS   // időalapú fázisléptetés

  cx.clearRect(0, 0, cv.width, cv.height)
  for (let i = 0; i < BUCKETS; i++) buckets[i].length = 0
  for (const s of stars) {
    s.p += s.s * k
    const a = .25 + Math.abs(Math.sin(s.p)) * .6
    let bi = (a * BUCKETS) | 0
    if (bi >= BUCKETS) bi = BUCKETS - 1
    buckets[bi].push(s)
  }
  cx.fillStyle = '#fff'
  for (let i = 0; i < BUCKETS; i++) {
    const b = buckets[i]
    if (!b.length) continue
    cx.globalAlpha = (i + .5) / BUCKETS
    for (let j = 0; j < b.length; j++) { const s = b[j]; cx.fillRect(s.x, s.y, s.r, s.r) }
  }
  cx.globalAlpha = 1
}

onMounted(() => {
  cv = document.getElementById('gbg-stars')
  if (!cv) return
  cx = cv.getContext('2d')
  if (!cx) return
  seed()
  onResize = () => { clearTimeout(resizeTimer); resizeTimer = setTimeout(() => { seed(); if (!running) draw(performance.now()) }, 150) }
  onVis = () => { document.hidden ? stop() : start() }
  addEventListener('resize', onResize)
  document.addEventListener('visibilitychange', onVis)
  start()
})

onBeforeUnmount(() => {
  stop()
  clearTimeout(resizeTimer)
  if (onResize) removeEventListener('resize', onResize)
  if (onVis) document.removeEventListener('visibilitychange', onVis)
})
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
/* A ködfoltok korábban filter:blur(110px)-et kaptak: két ~1000px-es réteget
   kellett volna minden keretben újra-elmosni. A gradiens önmagában is lágy
   lefutású, ezért a homályosságot több színátmenet-ponttal állítjuk elő —
   ugyanaz a látvány, számítás nélkül, és a réteg csak egyszer festődik. */
.gbg-nebula{position:absolute;border-radius:50%;opacity:.14;contain:strict}
.gbg-nebula.n1{width:55vw;height:55vw;left:-18vw;top:-22vw;
  background:radial-gradient(circle,#23355e 0%,rgba(35,53,94,.72) 26%,rgba(35,53,94,.38) 46%,rgba(35,53,94,.14) 64%,transparent 78%)}
.gbg-nebula.n2{width:50vw;height:50vw;right:-16vw;bottom:-20vw;
  background:radial-gradient(circle,#0d3a44 0%,rgba(13,58,68,.72) 26%,rgba(13,58,68,.38) 46%,rgba(13,58,68,.14) 64%,transparent 78%)}
</style>
