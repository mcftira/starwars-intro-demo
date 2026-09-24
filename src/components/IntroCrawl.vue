<script setup>
import { onBeforeUnmount, ref } from 'vue'
import { useSlideContext } from '@slidev/client'

const { $nav } = useSlideContext()

const stage = ref('gate') // gate -> film -> blue -> logo -> crawl
const muted = ref(false)
const hasVideo = ref(true)
/* A video src-t csak a gombra kattintva kotjuk be: igy a hianyzó
   video/intro.mp4 nem general felesleges 404-et minden oldalbetolteskor. */
const videoSrc = ref(null)
const timers = []
let audio = null
let video = null
let raf = 0

// Eredeti videó: 0:00–0:22 látható, utána csak a hangja fut tovább.
// Saját intro a videó órájához szinkronizálva (mp, a videó currentTime-ja):
const T = { film: 22, blue: 27, logo: 35, crawl: 113 }
// Fallback (fanfare.mp3), ha nincs videó:
const F = { blue: 0, logo: 5000, crawl: 13000, done: 95000 }

function later(fn, ms){ timers.push(setTimeout(fn, ms)) }

function start(){
  videoSrc.value = 'video/intro.mp4'
  video = document.getElementById('intro-film')
  if (video && hasVideo.value){
    video.volume = .9
    stage.value = 'film'
    const p = video.play()
    if (p && p.catch) p.catch(()=>{ if (stage.value === 'film') fallback() })
    raf = requestAnimationFrame(tick)
  } else {
    fallback()
  }
}

function fallback(){
  // nincs videó -> régi, fanfaros verzió
  if (video){ video.removeAttribute('src'); video = null }
  hasVideo.value = false
  stage.value = 'blue'
  audio = new Audio('audio/theme.mp3')
  audio.loop = true
  audio.volume = .5   // 50%: a felhasználó kérése szerint halkan
  audio.play().catch(()=>{})
  later(()=>{ stage.value = 'logo' }, F.logo)
  later(()=>{ stage.value = 'crawl' }, F.crawl)
  later(()=>{ done() }, F.done)
}

function tick(){
  if (!video) return
  const t = video.currentTime
  if (t >= T.crawl || video.ended){ done(); return }
  if (t >= T.logo && stage.value !== 'crawl') stage.value = 'crawl'
  else if (t >= T.blue && (stage.value === 'film' || stage.value === 'blue')) stage.value = 'logo'
  else if (t >= T.film && stage.value === 'film') stage.value = 'blue'
  raf = requestAnimationFrame(tick)
}

function onVideoError(){
  if (stage.value === 'gate'){ hasVideo.value = false }
  else fallback()
}

function fadeOut(){
  if (audio){
    const iv = setInterval(()=>{
      audio.volume = Math.max(0, audio.volume - .1)
      if (audio.volume <= 0){ audio.pause(); clearInterval(iv) }
    }, 100)
  }
  if (video){
    const iv = setInterval(()=>{
      video.volume = Math.max(0, video.volume - .15)
      if (video.volume <= 0){ video.pause(); clearInterval(iv) }
    }, 60)
  }
}

function done(){
  cancelAnimationFrame(raf)
  timers.forEach(clearTimeout)
  fadeOut()
  $nav.value.nextSlide()
}

function toggleMute(e){
  e.stopPropagation()
  const m = !(audio ? audio.muted : video ? video.muted : false)
  if (audio) audio.muted = m
  if (video) video.muted = m
  muted.value = m
}

onBeforeUnmount(()=>{
  cancelAnimationFrame(raf)
  timers.forEach(clearTimeout)
  if (audio){ audio.pause(); audio = null }
  if (video){ video.pause(); video = null }
})
</script>

<template>
  <div class="crawl-root">

    <!-- eredeti film: 0:00–0:22 látható, utána vizuálisan eltűnik, a hangja megy tovább -->
    <video
      v-show="hasVideo && stage==='film'"
      id="intro-film"
      class="film"
      :src="videoSrc"
      preload="auto"
      playsinline
      @error="onVideoError"
    ></video>

    <!-- start gate -->
    <div v-if="stage==='gate'" class="layer gate" @click.stop="start">
      <div class="gate-ring">
        <svg viewBox="0 0 24 24"><path d="M8 5v14l11-7z"/></svg>
      </div>
      <p>Kattints az indításhoz — hanggal</p>
    </div>

    <!-- blue card -->
    <transition name="lp-fade">
      <div v-if="stage==='blue'" class="layer">
        <p class="blue">Régen, egy nagyon távoli klaszterben&hellip;</p>
      </div>
    </transition>

    <!-- logo zoom -->
    <div v-if="stage==='logo' || stage==='crawl'" class="layer logo" :class="{ zoom: stage==='logo', gone: stage==='crawl' }">
      <svg viewBox="0 0 900 340" class="logo-svg">
        <text x="450" y="140" text-anchor="middle" class="lg">OPENSHIFT</text>
        <text x="450" y="300" text-anchor="middle" class="lg">LIGHTSPEED</text>
      </svg>
    </div>

    <!-- crawl -->
    <div v-if="stage==='crawl'" class="layer crawl-stage">
      <div class="crawl-plane">
        <div class="crawl-inner">
          <div class="episode">Episode 4.x</div>
          <h1>A fénysebesség kora</h1>
          <p>Zűrzavar uralkodik a galaxis klasztereiben. A podok <span class="em">CrashLoopBackOff</span> állapotba zuhantak, a deploymenteknél hiányzik a magas rendelkezésre állás, a logok pedig szétszóródtak a replikák között.</p>
          <p>A sötét oldal fegyvere a kilencszáz oldalas dokumentáció és a végtelen <span class="em">kubectl debug</span>. Egyetlen remény maradt: az <span class="em">OPENSHIFT LIGHTSPEED</span> — helyi, offline mesterséges intelligencia, amely a Red Hat tudásbázisával válaszol, anélkül, hogy egyetlen bit is elhagyná a klasztert.</p>
          <p>A <span class="em">PLATFORM FÓRUM</span> hősei most bemutatják az architektúrát, és élő demóban bizonyítják: az Erő&hellip; izé, a <span class="em">RAG</span> velünk van&hellip;</p>
        </div>
      </div>
    </div>

    <button v-if="stage!=='gate'" class="nav-btn mute" @click="toggleMute">♪ hang: {{ muted ? 'ki' : 'be' }}</button>
    <button v-if="stage!=='gate'" class="nav-btn skip" @click.stop="done">Ugrás a diákra →</button>
  </div>
</template>

<style scoped>
.crawl-root{position:absolute;inset:0;background:transparent;overflow:hidden}
.layer{position:absolute;inset:0;display:flex;align-items:center;justify-content:center}

.film{position:absolute;inset:0;width:100%;height:100%;object-fit:cover;z-index:5;background:#000}

.gate{cursor:pointer;flex-direction:column;gap:26px;z-index:6}
.gate-ring{position:relative;width:120px;height:120px;border:2px solid var(--lp-yellow);border-radius:50%;
  display:flex;align-items:center;justify-content:center}
/* A lukteto halo korabban box-shadow-t animalt -> minden keretben ujrafestes.
   Most egy kulon retegen egy pseudo-elem attereszodik es nyulik, amit a
   kompozitor vegez: ugyanaz a latvany, festes nelkul. */
.gate-ring::after{content:"";position:absolute;inset:-2px;border-radius:50%;
  box-shadow:0 0 0 22px rgba(255,232,31,.35);opacity:0;pointer-events:none;
  animation:pulse 2s ease-out infinite;will-change:opacity,transform}
.gate-ring svg{width:44px;height:44px;fill:var(--lp-yellow);margin-left:8px}
.gate p{font-family:var(--lp-mono);font-size:13px;letter-spacing:.35em;color:var(--lp-yellow);text-transform:uppercase}
@keyframes pulse{0%{opacity:.9;transform:scale(.82)}70%{opacity:0;transform:scale(1.18)}100%{opacity:0;transform:scale(1.18)}}

.blue{font-family:var(--lp-crawl);color:#4bd5ee;font-weight:600;font-size:38px;letter-spacing:.06em;text-align:center;padding:0 8vw;line-height:1.5}
.lp-fade-enter-active,.lp-fade-leave-active{transition:opacity 1.2s ease}
.lp-fade-enter-from,.lp-fade-leave-to{opacity:0}

.logo{z-index:4}
.logo.gone{display:none}
.logo-svg{width:min(72%,820px);overflow:visible}
.logo-svg .lg{font-family:var(--lp-crawl);font-weight:900;font-size:150px;letter-spacing:.02em;
  fill:transparent;stroke:var(--lp-yellow);stroke-width:2.5;
  stroke-dasharray:1400;stroke-dashoffset:1400;animation:draw 2.6s ease forwards}
@keyframes draw{to{stroke-dashoffset:0}}
.logo.zoom{animation:logoZoom 11s cubic-bezier(.2,.6,.35,1) forwards}
@keyframes logoZoom{0%{transform:scale(2.4);opacity:0}8%{opacity:1}100%{transform:scale(.02);opacity:0}}

.crawl-stage{z-index:3;overflow:hidden;perspective:38vh;perspective-origin:50% 34%;
  -webkit-mask-image:linear-gradient(180deg,transparent 0%,#000 34%);
  mask-image:linear-gradient(180deg,transparent 0%,#000 34%)}
.crawl-plane{position:absolute;left:50%;bottom:0;width:min(86%,920px);
  transform-origin:50% 100%;transform:translateX(-50%) rotateX(57deg)}
.crawl-inner{animation:scroll 78s linear forwards;will-change:transform;backface-visibility:hidden}
@keyframes scroll{from{transform:translateY(105%)}to{transform:translateY(-340%)}}
.episode{font-family:var(--lp-crawl);font-weight:700;font-size:22px;letter-spacing:.3em;text-align:center;
  color:var(--lp-yellow);margin-bottom:1.6em;text-transform:uppercase}
.crawl-inner h1{font-family:var(--lp-crawl);font-size:52px;font-weight:900;text-align:center;color:var(--lp-yellow);
  letter-spacing:.12em;margin-bottom:1.4em;line-height:1.25;text-transform:uppercase}
.crawl-inner p{font-family:var(--lp-crawl);font-size:30px;font-weight:700;color:var(--lp-yellow);text-align:justify;
  line-height:1.55;margin-bottom:1.8em}
.crawl-inner .em{color:#fff}

.nav-btn{position:absolute;bottom:24px;z-index:7;font-family:var(--lp-mono);font-size:12px;
  letter-spacing:.25em;color:var(--lp-dim);background:none;border:1px solid var(--lp-border);
  padding:10px 18px;cursor:pointer;text-transform:uppercase;transition:color .2s,border-color .2s}
.nav-btn:hover{color:var(--lp-yellow);border-color:var(--lp-yellow)}
.nav-btn.skip{right:28px}
.nav-btn.mute{left:28px}

/* Mozgaservenyes felhasznalok: a csuszó szoveg es a logo-zoom helyett alló kep. */
@media (prefers-reduced-motion: reduce){
  .crawl-inner{animation:none;transform:translateY(-40%)}
  .logo.zoom{animation:none;transform:scale(1);opacity:1}
  .gate-ring::after{animation:none;opacity:.35}
}
</style>