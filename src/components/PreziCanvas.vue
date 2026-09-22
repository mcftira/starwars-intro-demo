<script setup>
import { computed, onMounted, onBeforeUnmount, ref } from 'vue'
import { useSlideContext } from '@slidev/client'

const { $clicks } = useSlideContext()

// Demo content also drives the summary cards and camera stops.
const demos = [
  {
    title: 'Klaszterverzió', summary: 'OpenShift és Kubernetes verzió',
    question: 'Mi a klaszter verziója (OpenShift és Kubernetes)?',
    principle: 'Élő klaszteradatból azonosítja az aktuális állapotot, majd közérthetően összefoglalja.',
    flow: [
      ['Felismerés', 'Azonosítja a platform és az orchesztrátor aktuális verzióját.'],
      ['Javaslat', 'Kontextusba helyezi a kompatibilitási és frissítési lehetőségeket.'],
      ['Ellenőrzés', 'Összeveti a verziókat a klaszter tényleges állapotával.'],
    ],
    x: 8000, y: 3300,
  },
  {
    title: 'Init:CrashLoopBackOff', summary: 'Hiba feltárása és javítása',
    question: 'Miért került a pod Init:CrashLoopBackOff állapotba?',
    followup: 'Végezd el a javítást a deployment-ben.',
    principle: 'A tünettől eljut a kiváltó okig, majd biztonságos javítási irányt ad.',
    flow: [
      ['Felismerés', 'Feltárja, melyik init lépés és milyen ok miatt hiúsul meg.'],
      ['Javaslat', 'A kiváltó okhoz illeszkedő, minimális javítást ajánl.'],
      ['Ellenőrzés', 'Visszaigazolja a sikeres inicializálást és az egészséges rolloutot.'],
    ],
    x: 4800, y: 3300,
  },
  {
    title: 'HA deployment', summary: 'Podok elosztása két adatközpont között',
    question: 'Hogyan lehetne HA a podhoz kapcsolódó deployment?',
    followup: 'TopologySpreadConstraints-szel oldjuk meg; a topologyKey a node-on lévő site label legyen. Amennyire lehet, egyenletesen osszuk szét a podokat a két adatközpont között.',
    principle: 'A rendelkezésre állási célt a klaszter topológiájához illesztett elhelyezési szabállyá fordítja.',
    flow: [
      ['Felismerés', 'Felméri az aktuális példányszámot és a topológiai kockázatokat.'],
      ['Javaslat', 'Egyenletes, site-tudatos pod-elhelyezési elvet alakít ki.'],
      ['Ellenőrzés', 'Megvizsgálja, hogy a podok valóban a kívánt módon oszlanak-e el.'],
    ],
    x: 1600, y: 3300,
  },
  {
    title: 'Aggregált logok', summary: 'Loggyűjtő, verzió és konzolos elérés',
    question: 'Van a klaszterben telepített loggyűjtő, amivel meg tudom nézni aggregáltan a deployment összes példányának a logját? Ha van, akkor mi a verziója és a konzolon hol érem el?',
    followup: 'Kérek egy PromQL-t, ami megjeleníti a pod nevét és a message-t.',
    principle: 'Felderíti az elérhető megfigyelhetőségi képességeket, és egységes keresési utat mutat.',
    flow: [
      ['Felismerés', 'Felderíti a telepített loggyűjtési és keresési képességeket.'],
      ['Javaslat', 'Egységes lekérdezési utat ad az alkalmazás összes példányához.'],
      ['Ellenőrzés', 'Igazolja, hogy minden replika logja azonosítható és kereshető.'],
    ],
    x: 1600, y: 5500,
  },
  {
    title: 'Autoscaler', summary: 'Skálázás és egyedi metrikák',
    question: 'Az alkalmazáshoz hogyan állítsak be autoscaler-t? Van olyan telepített megoldás, amiben egyedi metrika alapján skáláz a rendszer?',
    principle: 'A terhelési jel és a telepített képességek alapján megfelelő skálázási stratégiát választ.',
    flow: [
      ['Felismerés', 'Felméri az alkalmazás erőforrásait, metrikáit és skálázási képességeit.'],
      ['Javaslat', 'A terhelési jelhez illő skálázási modellt választ.'],
      ['Ellenőrzés', 'Visszaméri, hogy a replika-szám a kívánt jel alapján változik-e.'],
    ],
    x: 4800, y: 5500,
  },
  {
    title: 'Ingress megoldások', summary: 'Controllerek és támogatott annotációk',
    question: 'Milyen ingress kiszolgáló megoldások érhetők el a klaszterben? Mik a támogatott ingress annotációk az egyes ingress-controller-eknél?',
    principle: 'A klaszter leltárát és a támogatási tudást összekapcsolva összehasonlíthatóvá teszi a lehetőségeket.',
    flow: [
      ['Felismerés', 'Azonosítja az elérhető controllereket és azok támogatott képességeit.'],
      ['Javaslat', 'Az alkalmazás igényeihez illő ingress megoldást választ.'],
      ['Ellenőrzés', 'Igazolja az elérhetőséget és a választott beállítás támogatottságát.'],
    ],
    x: 8000, y: 5500,
  },
]

/* Camera stops: x, y, zoom, rotation (degrees), label. */
const W = [
  { x: 1600,  y: 1100, z: 0.58, r: 0,  label: 'OpenShift Lightspeed' },
  { x: 6400,  y: 4400, z: 0.15, r: 0,  label: 'Áttekintés' },
  { x: 4800,  y: 1100, z: 0.62, r: 0,  label: 'Agenda' },
  { x: 8000,  y: 1100, z: 0.42, r: 0,  label: 'Architektúra high level' },
  { x: 11200, y: 1100, z: 0.62, r: -2, label: 'Miért helyi LLM?' },
  { x: 11200, y: 3300, z: 0.56, r: 0,  label: 'Demó — hat kérdés' },
  ...demos.map((demo, i) => ({
    x: demo.x, y: demo.y, z: 0.60, r: 0,
    label: `UC-${String(i + 1).padStart(2, '0')} · ${demo.title}`,
  })),
  { x: 11200, y: 5500, z: 0.54, r: -2, label: 'Továbbfejlesztés' },
  { x: 11200, y: 7700, z: 0.56, r: 0,  label: 'Összegzés + Q&A' },
  { x: 6400,  y: 4400, z: 0.15, r: 0,  label: 'Köszönjük!' },
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

const OVERVIEW_IDX = new Set([1, W.length - 1])
const camStyle = computed(() => {
  const c = current.value
  // Fit all four rows, leaving room for the HUD.
  const z = OVERVIEW_IDX.has(idx.value)
    ? Math.min(vw.value / 12600, (vh.value - 64) / 8800) * 0.94
    : c.z
  return {
    transform:
      `translate(${vw.value/2}px, ${vh.value/2}px) rotate(${-c.r}deg) scale(${z}) translate(${-c.x}px, ${-c.y}px)`,
  }
})

/* ---------- path (Prezi connector line) ---------- */
const route = W.filter((_, i) => !OVERVIEW_IDX.has(i))
const pathPoints = route.map(p => `${p.x},${p.y}`).join(' ')

</script>

<template>
  <div ref="root" class="pz-root">

    <!-- ======= WORLD ======= -->
    <div class="pz-world" :style="camStyle">
      <!-- connector path -->
      <svg class="pz-path" viewBox="0 0 12800 8800">
        <polyline :points="pathPoints" fill="none" stroke="#6ea8d8" stroke-width="4"
          stroke-dasharray="14 18" opacity="0.35" stroke-linejoin="round"/>
        <circle v-for="(p,i) in route" :key="i" :cx="p.x" :cy="p.y" r="14"
          :fill="p===current ? '#e8b23a' : '#6ea8d8'" opacity="0.85"/>
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
      <section class="pz-sec" :class="{ next: idx===2 }" style="left:4800px; top:1100px; width:1500px;">
        <div class="lp-panel">
          <div class="lp-kicker" style="font-size:22px;"><span class="idx">01 /</span> Agenda</div>
          <h3 style="font-size:60px; margin:18px 0 26px;">Ma három <span class="accent">csillagrendszer</span> járunk be</h3>
          <ul class="lp-points" style="font-size:29px;">
            <li><b>Architektúra high level</b> — helyi LLM, Red Hat offline knowledge base, mi fut és hol</li>
            <li><b>Élő demó</b> — hat use-case: klaszterverzió, Init:CrashLoopBackOff, HA, aggregált logok, autoscaler és ingress</li>
            <li><b>Továbbfejlesztés</b> — OTP-specifikus tudás, klaszter-introspekció, okos modellválasztás</li>
            <li class="dim">Felosztás: egyikünk az architektúrát mutatja be, másikunk a demót vezeti</li>
          </ul>
        </div>
      </section>

      <!-- ARCHITEKTÚRA -->
      <section class="pz-sec" :class="{ next: idx===3 }" style="left:8000px; top:1100px; width:2500px;">
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
      <section class="pz-sec" :class="{ next: idx===4 }" style="left:11200px; top:1100px; width:1500px;">
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
      <section class="pz-sec" :class="{ next: idx===5 }" style="left:11200px; top:3300px; width:1700px;">
        <div class="lp-panel demo-overview">
          <div class="lp-kicker"><span class="idx">04 /</span> Élő demó</div>
          <h3>Hat kérdés, <span class="accent">élő klaszterkontextus</span></h3>
          <div class="demo-grid">
            <div v-for="(demo, i) in demos" :key="demo.title" class="lp-uc demo-card">
              <span class="no">UC-{{ String(i + 1).padStart(2, '0') }}</span>
              <h4>{{ demo.title }}</h4>
              <p>{{ demo.summary }}</p>
            </div>
          </div>
        </div>
      </section>

      <!-- One camera stop per query; follow-ups stay on the same panel. -->
      <section v-for="(demo, i) in demos" :key="demo.title"
        class="pz-sec" :class="{ next: idx===i+6 }"
        :style="{ left: `${demo.x}px`, top: `${demo.y}px`, width: '1700px' }">
        <div class="lp-panel demo-panel">
          <div class="lp-kicker"><span class="idx">{{ String(i + 5).padStart(2, '0') }} /</span> Demó · UC-{{ String(i + 1).padStart(2, '0') }}</div>
          <h3>{{ demo.title }}</h3>
          <div class="demo-query">
            <div class="demo-caption">Kérdés</div>
            <p>{{ demo.question }}</p>
          </div>
          <div v-if="demo.followup" class="demo-followup">
            <div class="demo-caption">Következő kérés</div>
            <p>{{ demo.followup }}</p>
          </div>
          <div class="demo-principle">
            <div class="demo-caption">Mit mutatunk meg?</div>
            <p>{{ demo.principle }}</p>
          </div>
          <div class="demo-flow" aria-label="A demó folyamata">
            <div v-for="(step, stepIndex) in demo.flow" :key="step[0]">
              <div class="demo-flow-title"><span>0{{ stepIndex + 1 }}</span>{{ step[0] }}</div>
              <p>{{ step[1] }}</p>
            </div>
          </div>
        </div>
      </section>

      <!-- TOVÁBBFEJLESZTÉS -->
      <section class="pz-sec" :class="{ next: idx===12 }" style="left:11200px; top:5500px; width:1700px;">
        <div class="lp-panel">
          <div class="lp-kicker" style="font-size:22px;"><span class="idx">11 /</span> Továbbfejlesztés</div>
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
      <section class="pz-sec" :class="{ next: idx===13 }" style="left:11200px; top:7700px; width:1600px;">
        <div class="lp-panel">
          <div class="lp-kicker" style="font-size:22px;"><span class="idx">12 /</span> Összegzés</div>
          <h3 style="font-size:52px; margin:16px 0 24px;">A klaszter <span class="accent">tudja a választ</span> — végre meg is kérdezhetjük</h3>
          <ul class="lp-points" style="font-size:28px;">
            <li><b>Helyi LLM + offline Red Hat tudásbázis</b> = AI-asszisztens banki környezetre</li>
            <li><b>A demó hat kérdése</b> a ti mindennapjaitokból jött</li>
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

.pz-world{position:absolute;left:0;top:0;width:12800px;height:8800px;transform-origin:0 0;
  transition:transform 1.5s cubic-bezier(.22,.61,.36,1);will-change:transform}
.pz-path{position:absolute;left:0;top:0;width:12800px;height:8800px;pointer-events:none}
.pz-sec{position:absolute;transform:translate(-50%,-50%);transition:filter .8s}
.pz-sec.next .lp-panel{animation:nextGlow 2s infinite}
@keyframes nextGlow{0%,100%{box-shadow:0 18px 60px rgba(0,0,0,.55),0 0 0 0 rgba(232,178,58,0)}
  50%{box-shadow:0 18px 60px rgba(0,0,0,.55),0 0 44px 4px rgba(232,178,58,.28)}}

.demo-overview .lp-kicker,.demo-panel .lp-kicker{font-size:22px}
.demo-overview h3,.demo-panel h3{font-size:54px;margin:18px 0 30px}
.demo-grid{display:grid;grid-template-columns:1fr 1fr;gap:22px}
.demo-card{padding:30px 34px}
.demo-card .no{position:static;display:block;font-size:18px;margin-bottom:12px}
.demo-card h4{font-family:var(--lp-display);font-size:32px;font-weight:700;margin:0 0 10px;color:var(--lp-text)}
.demo-card p{font-size:23px;line-height:1.4;margin:0;color:var(--lp-dim)}
.demo-panel{padding:48px 56px}
.demo-query,.demo-followup{padding:24px 30px;border-radius:14px}
.demo-query{background:#0a0e14f2;border:1px solid var(--lp-border);border-left:5px solid var(--lp-steel)}
.demo-followup{margin-top:20px;background:var(--lp-glass-flat);border:1px solid var(--lp-border);border-left:5px solid var(--lp-gold)}
.demo-caption{font-family:var(--lp-mono);font-size:18px;letter-spacing:.2em;text-transform:uppercase;color:var(--lp-steel);margin-bottom:14px}
.demo-followup .demo-caption{color:var(--lp-gold)}
.demo-query p{font-size:34px;line-height:1.4;margin:0;overflow-wrap:anywhere}
.demo-followup p{font-size:28px;line-height:1.4;margin:0;overflow-wrap:anywhere}
.demo-principle{display:grid;grid-template-columns:260px 1fr;align-items:center;gap:28px;margin-top:22px;padding:20px 28px;border-top:1px solid var(--lp-border)}
.demo-principle .demo-caption{margin:0;color:var(--lp-gold)}
.demo-principle p{font-size:25px;line-height:1.4;margin:0;color:var(--lp-text)}
.demo-flow{display:grid;grid-template-columns:repeat(3,1fr);gap:14px;margin-top:18px}
.demo-flow>div{padding:16px 18px;border:1px solid var(--lp-border);border-radius:10px;background:#0a0e1499}
.demo-flow-title{display:flex;align-items:center;gap:12px;font-family:var(--lp-mono);font-size:17px;letter-spacing:.08em;text-transform:uppercase;color:var(--lp-dim)}
.demo-flow-title span{color:var(--lp-gold)}
.demo-flow p{font-size:18px;line-height:1.35;margin:10px 0 0;color:var(--lp-text)}

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
