<script setup>
import { computed, onMounted, onBeforeUnmount, ref } from 'vue'
import { useSlideContext } from '@slidev/client'

const { $clicks } = useSlideContext()

/* ---------- waypoints: x, y, zoom, rotation(deg), label ----------
   Grid: oszlopok x = 1600 / 4800 / 8000 / 11200, sorok y = 1100 / 3300 / 5500.
   A szekciók center-anchorúak: a waypoint pontosan a panel közepe. */
const W = [
  { x: 1600,  y: 1100, z: 0.58, r: 0,   label: 'OpenShift Lightspeed' },
  { x: 6400,  y: 3300, z: 0.15, r: 0,   label: 'Áttekintés' },
  { x: 4800,  y: 1100, z: 0.62, r: 0,   label: 'Agenda' },
  { x: 8000,  y: 1100, z: 0.42, r: 0,   label: 'Architektúra high level' },
  { x: 11200, y: 1100, z: 0.62, r: -2,  label: 'Miért helyi LLM?' },
  { x: 11200, y: 3300, z: 0.56, r: 0,   label: 'Demó — use-case-ek' },
  { x: 8000,  y: 3300, z: 0.60, r: 0,   label: 'UC-01 · CrashLoop' },
  { x: 4800,  y: 3300, z: 0.60, r: 0,   label: 'UC-02 · HA deployment' },
  { x: 4800,  y: 5500, z: 0.56, r: 2,   label: 'UC-03 · Aggregált log' },
  { x: 8000,  y: 5500, z: 0.60, r: 0,   label: 'UC-04 · Autoscaler' },
  { x: 11200, y: 5500, z: 0.54, r: -2,  label: 'Továbbfejlesztés' },
  { x: 1600,  y: 3300, z: 0.56, r: 0,   label: 'Összegzés + Q&A' },
  { x: 6400,  y: 3300, z: 0.15, r: 0,   label: 'Köszönjük!' },
]

const idx = computed(() => Math.min(Math.max($clicks.value ?? 0, 0), W.length - 1))
const current = computed(() => W[idx.value])

/* ---------- viewport measurement (local slide coords) ---------- */
const root = ref(null)
const vw = ref(1200), vh = ref(675)
let ro = null
onMounted(() => {
  const el = root.value
  if (!el) return
  const measure = () => { vw.value = el.clientWidth; vh.value = el.clientHeight }
  measure()
  ro = new ResizeObserver(measure)
  ro.observe(el)
})
onBeforeUnmount(() => { ro?.disconnect() })

const OVERVIEW_IDX = new Set([1, 12])
const camStyle = computed(() => {
  const c = current.value
  // Áttekintés: a teljes tartalom-bbox (≈700..12050 × 500..6100) mindig férjen bele
  const z = OVERVIEW_IDX.has(idx.value)
    ? Math.min(vw.value / 12600, vh.value / 6500) * 0.94
    : c.z
  return {
    transform:
      `translate(${vw.value/2}px, ${vh.value/2}px) rotate(${-c.r}deg) scale(${z}) translate(${-c.x}px, ${-c.y}px)`,
  }
})

/* ---------- path (Prezi connector line) ---------- */
const route = [0,2,3,4,5,6,7,8,9,10,11].map(i => W[i])
const pathPoints = route.map(p => `${p.x},${p.y}`).join(' ')

</script>

<template>
  <div ref="root" class="pz-root">

    <!-- ======= WORLD ======= -->
    <div class="pz-world" :style="camStyle">
      <!-- connector path -->
      <svg class="pz-path" viewBox="0 0 12800 6600">
        <polyline :points="pathPoints" fill="none" stroke="#6ea8d8" stroke-width="4"
          stroke-dasharray="14 18" opacity="0.35" stroke-linejoin="round"/>
        <circle v-for="(p,i) in route" :key="i" :cx="p.x" :cy="p.y" r="14"
          :fill="i+1===idx || (i===0&&idx===0) ? '#e8b23a' : '#6ea8d8'" opacity="0.85"/>
      </svg>

      <!-- HERO -->
      <section class="pz-sec" style="left:1600px; top:1100px; width:1800px;">
        <div class="hero">
          <div class="lp-kicker" style="font-size:24px;">Platform Fórum · Tech demó</div>
          <h1 class="hero-title"><span class="rd">OpenShift</span> <span class="gd">Lightspeed</span></h1>
          <div class="hero-rule"></div>
          <div class="hero-sub lp-mono">Helyi LLM + offline Red Hat tudásbázis a klaszterben · Architektúra + élő demó</div>
        </div>
      </section>

      <!-- AGENDA -->
      <section class="pz-sec" :class="{ next: idx===1 }" style="left:4800px; top:1100px; width:1500px;">
        <div class="lp-panel">
          <div class="lp-kicker" style="font-size:22px;"><span class="idx">01 /</span> Agenda</div>
          <h3 style="font-size:60px; margin:18px 0 26px;">Ma három <span class="accent">csillagrendszer</span> járunk be</h3>
          <ul class="lp-points" style="font-size:29px;">
            <li><b>Architektúra high level</b> — helyi LLM, Red Hat offline knowledge base, mi fut és hol</li>
            <li><b>Élő demó</b> — négy use-case fejlesztőknek és üzemeltetőknek: CrashLoop, HA, aggregált log, autoscaler</li>
            <li><b>Továbbfejlesztés</b> — OTP-specifikus tudás, klaszter-introspekció, okos modellválasztás</li>
            <li class="dim">Felosztás: egyikünk az architektúrát mutatja be, másikunk a demót vezeti</li>
          </ul>
        </div>
      </section>

      <!-- ARCHITEKTÚRA -->
      <section class="pz-sec" :class="{ next: idx===2 }" style="left:8000px; top:1100px; width:2500px;">
        <div class="lp-panel" style="padding:48px 56px;">
          <div class="lp-kicker" style="font-size:22px;"><span class="idx">02 /</span> Architektúra high level</div>
          <h3 style="font-size:58px; margin:16px 0 30px;">Minden <span class="accent">a klaszteren belül</span> marad</h3>
          <div class="lp-arch" style="font-size:24px;">
            <div class="lp-anode">
              <h4 style="font-size:20px;">Kliens</h4>
              <div class="big" style="font-size:30px;">OpenShift webkonzol + Lightspeed plugin</div>
              <ul style="font-size:22px;">
                <li>Chat panel a konzolban, ahol dolgozol</li>
                <li><b>Klaszterkontextus</b> csatolva: projekt, erőforrás</li>
                <li>REST API — CLI-ből, CI-ból is hívható</li>
              </ul>
            </div>
            <div class="arrow"></div>
            <div class="lp-anode">
              <h4 style="font-size:20px;">Lightspeed Service · FastAPI</h4>
              <div class="big" style="font-size:30px;">Kérdés → validálás → RAG → válasz</div>
              <ul style="font-size:22px;">
                <li><b>Auth</b>: K8s RBAC-alapú hozzáférés</li>
                <li><b>Question validator</b>: csak OpenShift-téma</li>
                <li><b>Redactor</b>: regex-szűrés érzékeny adatra</li>
                <li><b>RAG retriever</b>: vektoros keresés a doksi + KB felett</li>
                <li>Conversation cache · token quota · Prometheus</li>
              </ul>
            </div>
            <div class="arrow"></div>
            <div class="lp-anode">
              <h4 style="font-size:20px;">LLM — helyi</h4>
              <div class="big" style="font-size:30px;">OpenShift AI + vLLM GPU-n</div>
              <ul style="font-size:22px;">
                <li>Pl. <b>Qwen3 / Granite</b>, token-auth route</li>
                <li>Alternatíva: RHEL AI, dev-ben <b>Ollama</b></li>
                <li><b>A kérdés és a klaszteradat nem megy ki</b></li>
              </ul>
            </div>
          </div>
          <div style="display:grid; grid-template-columns:1.35fr 1fr; gap:26px; margin-top:26px;">
            <div class="lp-subnode">
              <h4 style="font-size:18px;">Tudásbázis — offline</h4>
              <p style="font-size:21px;">Az OCP dokumentáció + Red Hat KB <b>tördelve, vektorosítva, lokálisan</b> — offline csomagból frissíthető, később saját tartalommal bővíthető (BYOK).</p>
            </div>
            <div class="lp-subnode">
              <h4 style="font-size:18px;">Konfiguráció</h4>
              <p style="font-size:21px;">Egyetlen <b>OLSConfig</b> CR: provider, modell, telemetria, adatgyűjtés ki/be.</p>
            </div>
          </div>
        </div>
      </section>

      <!-- MIÉRT LOKÁLIS -->
      <section class="pz-sec" :class="{ next: idx===3 }" style="left:11200px; top:1100px; width:1500px;">
        <div class="lp-panel">
          <div class="lp-kicker" style="font-size:22px;"><span class="idx">03 /</span> Miért helyi LLM?</div>
          <h3 style="font-size:58px; margin:16px 0 24px;">Banki környezetben <span class="accent">nem opcionális</span></h3>
          <ul class="lp-points" style="font-size:28px;">
            <li><b>Adatvédelem</b> — kérdések és klaszteradatok a hálózaton belül maradnak, nincs külső LLM API-hívás</li>
            <li><b>Telemetria kikapcsolható</b> — <span class="m">OLSConfig → userDataCollection</span></li>
            <li><b>Air-gapped működés</b> — image-ek <span class="m">oc mirror</span>-ral, KB offline csomagból</li>
            <li><b>Költség és kontroll</b> — nincs per-token díj, beépített token quota</li>
            <li><b>Egy GPU node elég a startba</b> — OpenShift AI + vLLM, később skálázható</li>
          </ul>
        </div>
      </section>

      <!-- DEMÓ ÁTTEKINTÉS -->
      <section class="pz-sec" :class="{ next: idx===4 }" style="left:11200px; top:3300px; width:1700px;">
        <div class="lp-panel">
          <div class="lp-kicker" style="font-size:22px;"><span class="idx">04 /</span> Élő demó</div>
          <h3 style="font-size:54px; margin:16px 0 26px;">Négy kérdés, <span class="accent">nulla dokumentáció-bányászat</span></h3>
          <div style="display:grid; grid-template-columns:1fr 1fr; gap:22px;">
            <div class="lp-uc"><span class="no" style="font-size:20px;">UC-01</span>
              <div class="q" style="font-size:27px;">Pod CrashLoop-ol, mi az oka?</div>
              <div class="tag" style="font-size:16px;">Téma: <b>hibaelhárítás</b> · üzemeltető</div></div>
            <div class="lp-uc"><span class="no" style="font-size:20px;">UC-02</span>
              <div class="q" style="font-size:27px;">Hogyan lehet HA ez a deployment?</div>
              <div class="tag" style="font-size:16px;">Téma: <b>architektúra</b> · fejlesztő</div></div>
            <div class="lp-uc"><span class="no" style="font-size:20px;">UC-03</span>
              <div class="q" style="font-size:27px;">Hol látom aggregálva az app összes példányának logját?</div>
              <div class="tag" style="font-size:16px;">Téma: <b>obszervabilitás</b> · mindkettő</div></div>
            <div class="lp-uc"><span class="no" style="font-size:20px;">UC-04</span>
              <div class="q" style="font-size:27px;">Hogyan állítsak be autoscaler-t?</div>
              <div class="tag" style="font-size:16px;">Téma: <b>skálázás</b> · fejlesztő</div></div>
          </div>
        </div>
      </section>

      <!-- DEMO 1 -->
      <section class="pz-sec" :class="{ next: idx===5 }" style="left:8000px; top:3300px; width:1500px;">
        <div class="lp-panel">
          <div class="lp-kicker" style="font-size:22px;"><span class="idx">05 /</span> Demó · UC-01</div>
          <div class="lp-quote" style="font-size:42px; margin:20px 0 6px;">Pod CrashLoop-ol, mi az oka?</div>
          <div class="lp-quote-who" style="font-size:18px;">Mindenki, hétfő reggel</div>
          <div class="lp-term" style="margin-top:26px;">
            <div class="lp-term-bar"><span class="dot r"></span><span class="dot y"></span><span class="dot g"></span><span class="ttl" style="font-size:16px;">lightspeed — konzol chat + webterminál</span></div>
            <div class="lp-term-body" style="font-size:22px;">
<div class="ln"><span class="cy">› Miért crashloopol a payment-service pod?</span></div>
<div class="ln"><span class="out">A CrashLoopBackOff leggyakoribb okai:</span></div>
<div class="ln"><span class="out">  1. OOMKilled (exit code 137) — memórialimit</span></div>
<div class="ln"><span class="out">  2. Sikertelen liveness/readiness probe</span></div>
<div class="ln">&nbsp;</div>
<div class="ln"><span class="ps">$ </span><span class="cm">oc describe pod payment-7d9f | grep -A3 "Last State"</span></div>
<div class="ln"><span class="out">    Reason: <span class="rd">OOMKilled</span> · Exit Code: 137</span></div>
<div class="ln"><span class="yl">→ lightspeed: emeld a limits.memory-t, igazítsd a JVM -Xmx-et</span> <span class="lp-cursor"></span></div>
            </div>
          </div>
          <ol class="lp-steps" style="font-size:24px; margin-top:22px;">
            <li>A kérdést <span class="m">magyarul</span> tesszük fel a konzol chatjében</li>
            <li>A javasolt parancsokat azonnal futtatjuk a webterminálban</li>
          </ol>
        </div>
      </section>

      <!-- DEMO 2 -->
      <section class="pz-sec" :class="{ next: idx===6 }" style="left:4800px; top:3300px; width:1500px;">
        <div class="lp-panel">
          <div class="lp-kicker" style="font-size:22px;"><span class="idx">06 /</span> Demó · UC-02</div>
          <div class="lp-quote" style="font-size:42px; margin:20px 0 6px;">Hogyan lehet HA ez a deployment?</div>
          <div class="lp-quote-who" style="font-size:18px;">Fejlesztő, go-live előtt egy nappal</div>
          <div class="lp-term" style="margin-top:26px;">
            <div class="lp-term-bar"><span class="dot r"></span><span class="dot y"></span><span class="dot g"></span><span class="ttl" style="font-size:16px;">lightspeed — generált YAML</span></div>
            <div class="lp-term-body" style="font-size:22px;">
<div class="ln"><span class="cy">› Mitől lesz magas rendelkezésre állású?</span></div>
<div class="ln"><span class="cm">spec:</span></div>
<div class="ln"><span class="cm">  replicas: <span class="yl">3</span>   <span class="out"># min. 2, ideálisan 3</span></span></div>
<div class="ln"><span class="cm">  template.spec.affinity:</span></div>
<div class="ln"><span class="cm">    podAntiAffinity: <span class="out"># podok külön node-ra</span></span></div>
<div class="ln">&nbsp;</div>
<div class="ln"><span class="ps">$ </span><span class="cm">oc apply -f pdb.yaml <span class="out"># minAvailable: 1</span></span></div>
<div class="ln"><span class="yl">→ lightspeed: PDB nélkül a node-drain mindent vihet</span> <span class="lp-cursor"></span></div>
            </div>
          </div>
          <ol class="lp-steps" style="font-size:24px; margin-top:22px;">
            <li>Checklist: replikák, <span class="m">podAntiAffinity</span>, PDB, probe-ok</li>
            <li>Élő bizonyíték: node-drain alatt is él a szolgáltatás</li>
          </ol>
        </div>
      </section>

      <!-- DEMO 3 -->
      <section class="pz-sec" :class="{ next: idx===7 }" style="left:4800px; top:5500px; width:1500px;">
        <div class="lp-panel">
          <div class="lp-kicker" style="font-size:22px;"><span class="idx">07 /</span> Demó · UC-03</div>
          <div class="lp-quote" style="font-size:38px; margin:20px 0 6px;">Hol látom aggregálva az app összes példányának logját?</div>
          <div class="lp-quote-who" style="font-size:18px;">Üzemeltető, incidens közben</div>
          <div class="lp-term" style="margin-top:26px;">
            <div class="lp-term-bar"><span class="dot r"></span><span class="dot y"></span><span class="dot g"></span><span class="ttl" style="font-size:16px;">lightspeed — log aggregáció</span></div>
            <div class="lp-term-body" style="font-size:22px;">
<div class="ln"><span class="cy">› Hol látom egy helyen az összes replika logját?</span></div>
<div class="ln"><span class="ps">$ </span><span class="cm">oc logs -l app=payment --prefix --since=30m | grep -i error</span></div>
<div class="ln"><span class="out">  [pod/payment-6f8a/api] ERROR tx-4815 timeout</span></div>
<div class="ln"><span class="out">  [pod/payment-9c2b/api] ERROR tx-4821 timeout</span></div>
<div class="ln">&nbsp;</div>
<div class="ln"><span class="out">Tartósan: Console → Observe → Logs (LokiStack):</span></div>
<div class="ln"><span class="cm">  {kubernetes_labels_app="payment"} |= "ERROR"</span></div>
<div class="ln"><span class="yl">→ lightspeed: a --prefix mutatja, melyik replika dobta</span> <span class="lp-cursor"></span></div>
            </div>
          </div>
          <ol class="lp-steps" style="font-size:24px; margin-top:22px;">
            <li>Gyors válasz CLI-ből: label-szelektor + <span class="m">--prefix</span></li>
            <li>Élőben szűrünk ERROR szintre az összes replikában</li>
          </ol>
        </div>
      </section>

      <!-- DEMO 4 -->
      <section class="pz-sec" :class="{ next: idx===8 }" style="left:8000px; top:5500px; width:1500px;">
        <div class="lp-panel">
          <div class="lp-kicker" style="font-size:22px;"><span class="idx">08 /</span> Demó · UC-04</div>
          <div class="lp-quote" style="font-size:42px; margin:20px 0 6px;">Hogyan állítsak be autoscaler-t?</div>
          <div class="lp-quote-who" style="font-size:18px;">Fejlesztő, kampánycsúcs előtt</div>
          <div class="lp-term" style="margin-top:26px;">
            <div class="lp-term-bar"><span class="dot r"></span><span class="dot y"></span><span class="dot g"></span><span class="ttl" style="font-size:16px;">lightspeed — HPA</span></div>
            <div class="lp-term-body" style="font-size:22px;">
<div class="ln"><span class="cy">› Állítsak be autoscalert a payment deploymentre.</span></div>
<div class="ln"><span class="ps">$ </span><span class="cm">oc autoscale deployment payment --min=3 --max=12 --cpu-percent=70</span></div>
<div class="ln"><span class="out">  horizontalpodautoscaler.autoscaling/payment autoscaled</span></div>
<div class="ln">&nbsp;</div>
<div class="ln"><span class="ps">$ </span><span class="cm">oc get hpa payment -w</span></div>
<div class="ln"><span class="out">  NAME     TARGETS   MIN  MAX  REPLICAS</span></div>
<div class="ln"><span class="out">  payment  84%/70%   3    12   <span class="yl">3 → 5 → 8</span></span></div>
<div class="ln"><span class="yl">→ lightspeed: egyéni metrikára (queue-hossz) KEDA-t javasol</span> <span class="lp-cursor"></span></div>
            </div>
          </div>
          <ol class="lp-steps" style="font-size:24px; margin-top:22px;">
            <li>Előfeltétel: CPU request — különben nincs mérőszám</li>
            <li>Élőben terhelünk, és nézzük, ahogy skáláz</li>
          </ol>
        </div>
      </section>

      <!-- TOVÁBBFEJLESZTÉS -->
      <section class="pz-sec" :class="{ next: idx===9 }" style="left:11200px; top:5500px; width:1700px;">
        <div class="lp-panel">
          <div class="lp-kicker" style="font-size:22px;"><span class="idx">09 /</span> Továbbfejlesztés</div>
          <h3 style="font-size:54px; margin:16px 0 24px;">Innen már csak a <span class="accent">saját tudásunk</span> hiányzik</h3>
          <ul class="lp-points" style="font-size:27px;">
            <li><b>OTP-specifikus tudás (BYOK RAG)</b> — saját runbookok, SecOps-folyamatok: <span class="m">„Hogyan lehet tűzfalat nyitni erre a pod-ra?”</span></li>
            <li><b>Klaszter-introspekció</b> — MCP-n élő klaszterállapot: nem általános recept, hanem a te podod állapota</li>
            <li><b>Okos modellválasztás</b> — vLLM Semantic Router (OCP 4.20): egyszerű kérdés kis modellhez, komplex a nagyhoz</li>
            <li><b>Redactor-szabályok</b> — belső regex-minták érzékeny adatokra</li>
            <li><b>Útiterv</b> — pilot tesztklaszteren → BYOK runbookok → produktív, telemetria nélküli üzem</li>
          </ul>
        </div>
      </section>

      <!-- ZÁRÁS -->
      <section class="pz-sec" :class="{ next: idx===10 }" style="left:1600px; top:3300px; width:1600px;">
        <div class="lp-panel">
          <div class="lp-kicker" style="font-size:22px;"><span class="idx">10 /</span> Összegzés</div>
          <h3 style="font-size:52px; margin:16px 0 24px;">A klaszter <span class="accent">tudja a választ</span> — végre meg is kérdezhetjük</h3>
          <ul class="lp-points" style="font-size:28px;">
            <li><b>Helyi LLM + offline Red Hat tudásbázis</b> = AI-asszisztens banki környezetre</li>
            <li><b>A demó négy kérdése</b> a ti mindennapjaitokból jött</li>
          </ul>
          <div class="lp-links lp-mono" style="font-size:22px; margin-top:30px; display:flex; flex-direction:column; gap:14px;">
            <div><span class="lbl" style="font-size:16px;">Szolgáltatás</span><a href="https://github.com/openshift/lightspeed-service">github.com/openshift/lightspeed-service</a></div>
            <div><span class="lbl" style="font-size:16px;">Dokumentáció</span><a href="https://docs.redhat.com/en/documentation/red_hat_openshift_lightspeed/">docs.redhat.com — OpenShift Lightspeed</a></div>
            <div><span class="lbl" style="font-size:16px;">Intro crawl</span><a href="https://github.com/KasselLabs/StarWarsIntroCreator">github.com/KasselLabs/StarWarsIntroCreator</a></div>
          </div>
          <div style="margin-top:34px; font-size:36px; font-weight:700; color:var(--lp-gold);">Az Erő… izé, a RAG legyen veletek. Kérdések?</div>
        </div>
      </section>
    </div>

    <!-- ======= HUD (viewport-fixed) ======= -->
    <div class="pz-hud">
      <div class="pz-label lp-mono">{{ String(idx+1).padStart(2,'0') }} · {{ current.label }}</div>
      <div class="pz-dots">
        <span v-for="(w,i) in W" :key="i" class="pz-dot" :class="{ on: i===idx }"></span>
      </div>
      <div class="pz-keys lp-mono">→ tovább · ← vissza</div>
    </div>
  </div>
</template>

<style scoped>
.pz-root{position:absolute;inset:0;background:transparent;overflow:hidden}

.pz-world{position:absolute;left:0;top:0;width:12800px;height:6600px;transform-origin:0 0;
  transition:transform 1.5s cubic-bezier(.22,.61,.36,1);will-change:transform}
.pz-path{position:absolute;left:0;top:0;width:12800px;height:6600px;pointer-events:none}
.pz-sec{position:absolute;transform:translate(-50%,-50%);transition:filter .8s}
.pz-sec.next .lp-panel{animation:nextGlow 2s infinite}
@keyframes nextGlow{0%,100%{box-shadow:0 18px 60px rgba(0,0,0,.55),0 0 0 0 rgba(232,178,58,0)}
  50%{box-shadow:0 18px 60px rgba(0,0,0,.55),0 0 44px 4px rgba(232,178,58,.28)}}

.hero{padding:40px}
.hero-title{font-family:var(--lp-display);font-size:118px;font-weight:900;letter-spacing:-.02em;line-height:1.04;margin:24px 0;white-space:nowrap}
.hero-title .rd{color:var(--lp-red)}
.hero-title .gd{color:var(--lp-gold)}
.hero-rule{width:150px;height:4px;background:var(--lp-red);margin-bottom:28px;border-radius:2px}
.hero-sub{font-size:27px;color:var(--lp-dim);letter-spacing:.06em}

.pz-hud{position:absolute;left:0;right:0;bottom:0;display:flex;align-items:center;gap:26px;
  padding:16px 26px;background:linear-gradient(180deg,transparent,#04060acc 65%);z-index:10}
.pz-label{font-size:13px;letter-spacing:.18em;color:var(--lp-steel);text-transform:uppercase;min-width:320px}
.pz-dots{flex:1;display:flex;gap:10px;justify-content:center}
.pz-dot{width:7px;height:7px;border-radius:50%;background:#2a3342;transition:all .3s}
.pz-dot.on{background:var(--lp-gold);transform:scale(1.5)}
.pz-keys{font-size:11px;letter-spacing:.15em;color:var(--lp-dim);text-transform:uppercase}
</style>
