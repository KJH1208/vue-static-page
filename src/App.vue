<template>
  <main class="wrap" role="main">
    <!-- background: hanji paper + gradient sky -->
    <div class="bg" aria-hidden="true"></div>

    <!-- header / hero -->
    <header class="hero">
      <div class="moon" :class="{ crescent: !isFullMoon }" aria-hidden="true">
        <div class="r1"/><div class="r2"/><div class="r3"/>
      </div>
      <div class="heading">
        <h1>한가위, 마음 가득 🎑</h1>
        <p class="sub">담백한 한지 카드 스타일의 Chuseok 페이지</p>
        <p class="today">오늘은 <strong>{{ dateText }}</strong> 입니다.</p>
      </div>
    </header>

    <!-- aligned action icons -->
    <nav class="rail" aria-label="동작">
      <button class="icon" @click="toggleMoon" :aria-pressed="isFullMoon.toString()">
        <svg viewBox="0 0 48 48" class="ico"><title>달 모양</title>
          <circle cx="24" cy="24" r="16" />
          <circle v-if="!isFullMoon" cx="16" cy="22" r="14" class="cut"/>
        </svg>
        <span>달</span>
      </button>
      <button class="icon" @click="focusInput" title="소원 적기">
        <svg viewBox="0 0 48 48" class="ico"><title>소원</title>
          <path d="M24 6c9.4 0 17 7.6 17 17s-7.6 17-17 17S7 32.4 7 23 14.6 6 24 6Z"/>
          <path class="cut" d="M15 24h18M24 15v18" />
        </svg>
        <span>소원</span>
      </button>
      <button class="icon" @click="shareOrCopy" title="공유">
        <svg viewBox="0 0 48 48" class="ico"><title>공유</title>
          <circle cx="12" cy="26" r="4"/><circle cx="34" cy="12" r="4"/><circle cx="36" cy="36" r="4"/>
          <path class="cut" d="M15 25l15-11M15 27l17 8"/>
        </svg>
        <span>공유</span>
      </button>
      <label class="icon toggle" :aria-pressed="dense.toString()">
        <input type="checkbox" v-model="dense" />
        <svg viewBox="0 0 48 48" class="ico" aria-hidden="true">
          <rect x="10" y="14" width="28" height="6" rx="3"/>
          <rect x="10" y="24" width="22" height="6" rx="3"/>
          <rect x="10" y="34" width="16" height="6" rx="3"/>
        </svg>
        <span>콤팩트</span>
      </label>
    </nav>

    <!-- card -->
    <section class="card" :class="{ dense }">
      <form class="wish" @submit.prevent="addWish">
        <input ref="wishInput" v-model.trim="draft" class="input" type="text" placeholder="소원을 적고 Enter" maxlength="80" aria-label="소원 입력" />
        <button class="btn" :disabled="!draft">적기</button>
      </form>

      <ul v-if="wishes.length" class="list" role="list">
        <li v-for="w in wishes" :key="w.id" class="item">
          <span class="txt">“{{ w.text }}”</span>
          <time class="ts" :datetime="w.ts.toISOString()">{{ formatTime(w.ts) }}</time>
          <button class="x" @click="removeWish(w.id)" aria-label="삭제">×</button>
        </li>
      </ul>

      <div class="actions">
        <button class="ghost" @click="clearAll" :disabled="!wishes.length">전체 삭제</button>
        <a class="ghost" :href="exportURL" download="wishes.json">내보내기</a>
      </div>
    </section>

    <!-- gentle lanterns (fewer, slower) -->
    <div class="lanterns" aria-hidden="true">
      <div v-for="lan in lanterns" :key="lan.id" class="lan" :style="lan.style"></div>
    </div>

    <footer class="credit">© {{ year }} Chuseok • Vue 3 + TS + Vite</footer>
  </main>
</template>

<script setup lang="ts">
import { computed, onMounted, reactive, ref, watch } from 'vue'

type Wish = { id: string; text: string; ts: Date }
const LS_KEY = 'chuseok:wishes:hanji'

const wishes = ref<Wish[]>([])
const draft = ref('')
const wishInput = ref<HTMLInputElement | null>(null)
const dense = ref(false)

function focusInput() { wishInput.value?.focus() }
function addWish() {
  if (!draft.value) return
  wishes.value.unshift({ id: crypto.randomUUID(), text: draft.value, ts: new Date() })
  draft.value = ''
}
function removeWish(id: string) { wishes.value = wishes.value.filter(w => w.id !== id) }
function clearAll() { if (wishes.value.length && confirm('모든 소원을 삭제할까요?')) wishes.value = [] }
function formatTime(d: Date) {
  try { return new Intl.DateTimeFormat('ko-KR', { month: 'short', day: 'numeric', hour: '2-digit', minute: '2-digit' }).format(d) } catch { return '' }
}

onMounted(() => {
  try {
    const raw = localStorage.getItem(LS_KEY)
    if (raw) wishes.value = (JSON.parse(raw) as { id: string; text: string; ts: string }[])\
  .map(w => ({ ...w, ts: new Date(w.ts) }))
  } catch {}
})
watch(wishes, v => {
  const safe = v.map(w => ({ ...w, ts: w.ts.toISOString() }))
  localStorage.setItem(LS_KEY, JSON.stringify(safe))
}, { deep: true })

async function shareOrCopy() {
  const text = '한가위 복 많이 받으세요 🎑'
  const url = location.href
  try {
    // @ts-expect-error share
    if (navigator.share) await navigator.share({ title: document.title || text, text, url })
    else { await navigator.clipboard.writeText(url); alert('링크 복사 완료!') }
  } catch {}
}

const dateText = computed(() => {
  const now = new Date()
  const fmt: Intl.DateTimeFormatOptions = { year: 'numeric', month: 'long', day: 'numeric', weekday: 'long' }
  return new Intl.DateTimeFormat('ko-KR', fmt).format(now)
})

const isFullMoon = ref(true)
function toggleMoon() { isFullMoon.value = !isFullMoon.value }
const themeHue = computed(() => isFullMoon.value ? 38 : 208)

// soft lantern particles
type Lantern = { id: number; style: Record<string, string> }
const lanterns = reactive<Lantern[]>([])
function rand(min: number, max: number) { return Math.random() * (max - min) + min }
function spawn(n = 8) {
  lanterns.length = 0
  for (let i = 0; i < n; i++) {
    const left = `${rand(0, 100)}%`
    const delay = `${rand(0, 6).toFixed(2)}s`
    const dur = `${rand(18, 28).toFixed(1)}s`
    const size = `${rand(8, 18).toFixed(0)}px`
    const hue = String(Math.floor(rand(20, 46)))
    lanterns.push({ id: i, style: { left, '--d': delay, '--t': dur, '--s': size, '--h': hue } })
  }
}
onMounted(() => spawn())

const year = new Date().getFullYear()

const exportURL = computed(() => {
  const arr = wishes.value.map(w => ({ id: w.id, text: w.text, ts: w.ts.toISOString() }))
  return 'data:application/json;charset=utf-8,' + encodeURIComponent(JSON.stringify(arr, null, 2))
})
</script>

<style scoped>
:root { color-scheme: dark; }
.wrap {
  --h: v-bind(themeHue);
  --paper: radial-gradient(1200px 900px at 70% 20%, rgba(255,255,255,.06), transparent 55%),
  radial-gradient(800px 600px at 20% 10%, rgba(255,255,255,.05), transparent 55%),
  linear-gradient(#0b1026, #1a2248 60%, #0b1026);
  --ink: #f7f9ff; --muted: #d6dbff;
  --accent: hsl(var(--h) 92% 60%);
  min-height: 100svh; display: grid; place-items: center; padding: clamp(16px,3vw,32px);
  font-family: ui-rounded, system-ui, -apple-system, "Apple SD Gothic Neo", "Noto Sans KR", "Segoe UI", Roboto, "Helvetica Neue", Arial, "Malgun Gothic", "맑은 고딕", sans-serif;
  position: relative; overflow: hidden; color: var(--ink);
}
.bg { position: absolute; inset: 0; background: var(--paper); z-index: -2; }

.hero { width: min(880px, 96%); display: grid; grid-template-columns: auto 1fr; gap: clamp(12px,3vw,20px); align-items: center; }
.moon { width: min(22vmin, 220px); aspect-ratio: 1; border-radius: 50%; position: relative; background:
    radial-gradient(circle at 35% 35%, rgba(255,255,255,.35), transparent 45%),
    radial-gradient(circle at 60% 60%, rgba(255,255,255,.2), transparent 46%),
    radial-gradient(circle at 50% 50%, #ffe7b0, #ffd36b 60%, #f3b94f 75%, #e6a23a 100%);
  box-shadow: 0 0 70px 20px rgba(255, 214, 132, .25), 0 0 160px 40px rgba(255, 214, 132, .12);
}
.moon.crescent::after { content: ""; position: absolute; inset: 0; border-radius: 50%; background: #0b1026; transform: translateX(18%); filter: blur(.2px); }
.moon .r1,.moon .r2,.moon .r3{ position:absolute; border-radius: 50%; opacity:.35; filter: blur(.4px); background: radial-gradient(circle at 35% 35%, rgba(0,0,0,.25), rgba(0,0,0,.12) 40%, transparent 60%);}
.moon .r1{ width:16%; aspect-ratio:1; left:22%; top:30%; }
.moon .r2{ width:10%; aspect-ratio:1; left:55%; top:52%; }
.moon .r3{ width:13%; aspect-ratio:1; left:45%; top:24%; }

.heading h1{ margin:0; font-size:clamp(1.6rem,3.6vw,2.4rem); font-weight:900; text-shadow:0 2px 18px rgba(255,255,255,.12); }
.heading .sub{ margin:.25rem 0 0; color: var(--muted); }
.heading .today{ margin:.25rem 0 0; color:#e5e9ff; }

/* rail */
.rail { width:min(880px,96%); margin:14px auto 10px; display:grid; grid-template-columns:repeat(4,1fr); gap:10px; }
.icon { display:grid; grid-template-rows:auto 1fr; justify-items:center; gap:6px; padding:10px 8px; border-radius:12px; background:rgba(255,255,255,.06); border:1px solid rgba(255,255,255,.16); color:var(--ink); cursor:pointer; }
.icon.toggle { cursor:default; }
.icon input{ position:absolute; opacity:0; pointer-events:none; }
.icon span{ font-size:.82rem; color: var(--muted); }
.ico{ width:28px; height:28px; fill:var(--accent); }
.cut{ fill:none; stroke:var(--ink); stroke-width:2.4; stroke-linecap:round; stroke-linejoin:round; opacity:.9; }

/* card */
.card{ width:min(880px,96%); padding: clamp(14px,3vw,22px); border-radius:16px; border:1px solid rgba(255,255,255,.16); background: linear-gradient(180deg, rgba(255,255,255,.10), rgba(255,255,255,.06)); box-shadow: 0 10px 40px rgba(0,0,0,.35); }
.card.dense .item{ padding:8px 10px; }

.wish{ display:grid; grid-template-columns:1fr auto; gap:10px; }
.input{ height:44px; padding:0 14px; border-radius:12px; outline:none; background: rgba(12,16,40,.55); border:1px solid rgba(255,255,255,.18); color:var(--ink); }
.input::placeholder{ color: color-mix(in oklab, var(--muted) 70%, #fff 30%); }
.btn{ height:44px; padding:0 16px; border-radius:12px; border:none; cursor:pointer; font-weight:800; color:#1a1a1a; background: linear-gradient(180deg, hsl(var(--h) 92% 62%), hsl(var(--h) 92% 54%)); box-shadow:0 6px 18px color-mix(in oklab, hsl(var(--h) 92% 62%) 35%, transparent); }
.btn:disabled{ opacity:.55; cursor:not-allowed; }

.list{ list-style:none; margin:12px 0 0; padding:0; display:grid; gap:8px; }
.item{ display:grid; grid-template-columns:1fr auto auto; gap:8px; align-items:center; padding:10px 12px; border-radius:12px; background: rgba(255,255,255,.06); border:1px solid rgba(255,255,255,.12); }
.txt{ color:#f6f8ff; }
.ts{ font-size:.8rem; color: var(--muted); }
.x{ width:28px; height:28px; border-radius:8px; border:1px solid rgba(255,255,255,.18); background: rgba(0,0,0,.25); color:var(--ink); cursor:pointer; font-size:16px; }

.actions{ display:flex; flex-wrap:wrap; gap:8px; justify-content:center; margin-top:12px; }
.ghost{ padding:10px 14px; border-radius:12px; cursor:pointer; border:1px solid rgba(255,255,255,.18); background: rgba(255,255,255,.06); color: var(--ink); }

/* soft lantern dots */
.lanterns{ position:absolute; inset:0; pointer-events:none; z-index:-1; }
.lan{ position:absolute; bottom:-40px; width:var(--s); height:var(--s); border-radius:50%; background: hsl(var(--h) 85% 60% / .9); box-shadow: 0 0 18px hsl(var(--h) 85% 60% / .55); animation: float var(--t) linear var(--d) infinite; }
@keyframes float { 0%{ transform: translateY(0); opacity:0 } 8%{opacity:.9} 92%{opacity:.9} 100%{ transform: translateY(-110vh); opacity:0 } }

.credit{ margin-top:14px; text-align:center; font-size:12px; color: color-mix(in oklab, var(--muted) 85%, #fff 15%); }

@media (max-width: 560px){ .hero{ grid-template-columns:1fr; justify-items:center; text-align:center; } .moon{ width:min(30vmin, 180px); } }
</style>