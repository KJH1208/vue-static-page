<!-- src/App.vue -->
<template>
  <main class="page" role="main">
    <!-- gradient sky -->
    <div class="sky" aria-hidden="true"></div>

    <!-- moon -->
    <div class="moon" aria-hidden="true">
      <div class="shadow s1"></div>
      <div class="shadow s2"></div>
      <div class="shadow s3"></div>
    </div>

    <!-- floating lanterns -->
    <div class="lanterns" aria-hidden="true">
      <div
          v-for="lan in lanterns"
          :key="lan.id"
          class="lantern"
          :style="lan.style"
      >
        <svg viewBox="0 0 64 96" class="lantern-svg" role="img">
          <title>등불</title>
          <defs>
            <linearGradient id="lanternGrad" x1="0" y1="0" x2="0" y2="1">
              <stop offset="0%" stop-opacity=".95" />
              <stop offset="100%" stop-opacity=".65" />
            </linearGradient>
          </defs>
          <path
              d="M32 2 C50 6 62 20 62 40 v30 c0 18-14 24-30 24S2 88 2 70V40C2 20 14 6 32 2z"
              fill="url(#lanternGrad)"
          />
          <ellipse cx="32" cy="74" rx="14" ry="8" class="lantern-glow" />
          <rect x="26" y="86" width="12" height="6" rx="2" class="lantern-knot" />
        </svg>
      </div>
    </div>

    <!-- card -->
    <section class="card container">
      <header class="stack center">
        <h1 class="title">한가위 잘 보내! 🌕</h1>
        <p class="subtitle">보름달처럼 마음도 넉넉해지길.</p>
        <p class="date" v-if="dateText">오늘은 <strong>{{ dateText }}</strong>야.</p>
      </header>

      <!-- aligned icon rail -->
      <nav class="icon-rail" aria-label="한가위 아이콘">
        <button class="icon-btn" @click="toggleMoon" :aria-pressed="isFullMoon.toString()">
          <svg viewBox="0 0 48 48" class="ico"><title>달 토글</title>
            <circle cx="24" cy="24" r="16" />
            <circle v-if="!isFullMoon" cx="18" cy="22" r="14" class="ico-cut"/>
          </svg>
          <span>달</span>
        </button>

        <button class="icon-btn" @click="focusInput" title="소원 적기">
          <svg viewBox="0 0 48 48" class="ico"><title>소원</title>
            <path d="M24 6c9.4 0 17 7.6 17 17s-7.6 17-17 17S7 32.4 7 23 14.6 6 24 6Z"/>
            <path class="ico-cut" d="M15 24h18M24 15v18" />
          </svg>
          <span>소원</span>
        </button>

        <button class="icon-btn" @click="shareOrCopy" title="공유하기">
          <svg viewBox="0 0 48 48" class="ico"><title>공유</title>
            <circle cx="12" cy="26" r="4"/>
            <circle cx="34" cy="12" r="4"/>
            <circle cx="36" cy="36" r="4"/>
            <path class="ico-cut" d="M15 25l15-11M15 27l17 8"/>
          </svg>
          <span>공유</span>
        </button>

        <button class="icon-btn" @click="toggleReduced" :aria-pressed="prefersReduced.toString()">
          <svg viewBox="0 0 48 48" class="ico"><title>애니메이션</title>
            <rect x="9" y="12" width="30" height="24" rx="5"/>
            <path class="ico-cut" d="M14 24h20" />
          </svg>
          <span>모션</span>
        </button>
      </nav>

      <!-- wish form -->
      <form class="wish-form" @submit.prevent="addWish">
        <input
            ref="wishInput"
            v-model.trim="draft"
            type="text"
            class="input"
            placeholder="소원을 적고 Enter!"
            maxlength="80"
            aria-label="소원 입력"
        />
        <button class="btn" :disabled="!draft">적기</button>
      </form>

      <!-- wishes -->
      <ul v-if="wishes.length" class="list" role="list">
        <li v-for="w in wishes" :key="w.id" class="item">
          <span class="txt">“{{ w.text }}”</span>
          <div class="row">
            <time class="ts" :datetime="w.ts.toISOString()">{{ formatTime(w.ts) }}</time>
            <button class="x" @click="removeWish(w.id)" aria-label="삭제">×</button>
          </div>
        </li>
      </ul>

      <div class="actions">
        <button class="ghost" @click="clearAll" :disabled="!wishes.length">전체 삭제</button>
        <a class="ghost" :href="exportURL" download="wishes.json">내보내기</a>
        <label class="check">
          <input type="checkbox" v-model="dense" />
          <span>콤팩트 보기</span>
        </label>
      </div>

      <footer class="credit">© {{ year }} Chuseok • Vue 3 + TS + Vite</footer>
    </section>
  </main>
</template>

<script setup lang="ts">
import { computed, onMounted, reactive, ref, watch } from 'vue'

type Wish = { id: string; text: string; ts: Date }
const LS_KEY = 'chuseok:wishes:v2'

const wishes = ref<Wish[]>([])
const draft = ref('')
const wishInput = ref<HTMLInputElement | null>(null)
const dense = ref(false)

function focusInput() {
  wishInput.value?.focus()
}

function addWish() {
  if (!draft.value) return
  wishes.value.unshift({ id: crypto.randomUUID(), text: draft.value, ts: new Date() })
  draft.value = ''
}

function removeWish(id: string) {
  wishes.value = wishes.value.filter(w => w.id !== id)
}

function clearAll() {
  if (!wishes.value.length) return
  if (confirm('모든 소원을 삭제할까?')) wishes.value = []
}

function formatTime(d: Date) {
  try {
    return new Intl.DateTimeFormat('ko-KR', {
      month: 'short', day: 'numeric',
      hour: '2-digit', minute: '2-digit'
    }).format(d)
  } catch { return '' }
}

/* storage */
onMounted(() => {
  try {
    const raw = localStorage.getItem(LS_KEY)
    if (raw) {
      const arr = JSON.parse(raw) as { id: string; text: string; ts: string }[]
      wishes.value = arr.map(w => ({ ...w, ts: new Date(w.ts) }))
    }
  } catch { /* ignore */ }
})
watch(wishes, v => {
  const safe = v.map(w => ({ ...w, ts: w.ts.toISOString() }))
  localStorage.setItem(LS_KEY, JSON.stringify(safe))
}, { deep: true })

/* share */
async function shareOrCopy() {
  const text = '한가위 복 많이 받아! 🌕'
  const url = location.href
  try {
    // @ts-expect-error web share
    if (navigator.share) await navigator.share({ title: document.title || text, text, url })
    else {
      await navigator.clipboard.writeText(url)
      alert('링크 복사 완료!')
    }
  } catch { /* canceled */ }
}

/* pretty date */
const dateText = computed(() => {
  const now = new Date()
  const fmt: Intl.DateTimeFormatOptions = { year: 'numeric', month: 'long', day: 'numeric', weekday: 'long' }
  return new Intl.DateTimeFormat('ko-KR', fmt).format(now)
})

/* icons / moon toggle (affects color theme) */
const isFullMoon = ref(true)
function toggleMoon() { isFullMoon.value = !isFullMoon.value }
const themeHue = computed(() => isFullMoon.value ? 42 : 210)

/* motion toggle (for users who want fewer animations) */
const prefersReduced = ref(
    matchMedia('(prefers-reduced-motion: reduce)').matches
)
function handlePrefers(e: MediaQueryListEvent) { prefersReduced.value = e.matches }
onMounted(() => {
  const mq = matchMedia('(prefers-reduced-motion: reduce)')
  mq.addEventListener?.('change', handlePrefers)
})
function toggleReduced() { prefersReduced.value = !prefersReduced.value }

/* lanterns */
type Lantern = { id: number; style: Record<string, string> }
const lanterns = reactive<Lantern[]>([])
function rand(min: number, max: number) { return Math.random() * (max - min) + min }
function spawnLanterns(n = 12) {
  lanterns.length = 0
  for (let i = 0; i < n; i++) {
    const left = `${rand(0, 100)}%`
    const delay = `${rand(0, 6).toFixed(2)}s`
    const duration = `${rand(12, 22).toFixed(1)}s`
    const scale = rand(0.8, 1.2).toFixed(2)
    const hue = String(Math.floor(rand(18, 48)))
    lanterns.push({ id: i, style: { left, '--delay': delay, '--dur': duration, '--scale': scale, '--hue': hue } })
  }
}
onMounted(() => spawnLanterns())

/* export wishes */
const exportURL = computed(() => {
  const blob = new Blob([JSON.stringify(wishes.value, null, 2)], { type: 'application/json' })
  return URL.createObjectURL(blob)
})

const year = new Date().getFullYear()
</script>

<style scoped>
/* ---------- design tokens ---------- */
:root { color-scheme: dark; }
.page {
  --h: v-bind(themeHue);
  --bg-1: #0a0f24;
  --bg-2: #15214a;
  --card: rgba(255,255,255,.07);
  --border: rgba(255,255,255,.16);
  --text: #f6f8ff;
  --muted: #cfd6ff;
  --accent: hsl(var(--h) 90% 62%);
  --accent-2: hsl(var(--h) 90% 54%);
  min-height: 100svh;
  display: grid;
  place-items: center;
  padding: clamp(16px, 3vw, 32px);
  font-family: ui-rounded, system-ui, -apple-system, "Apple SD Gothic Neo",
  "Noto Sans KR", "Segoe UI", Roboto, "Helvetica Neue", Arial, "Malgun Gothic", "맑은 고딕", sans-serif;
  overflow: hidden;
  position: relative;
}

/* ---------- background & moon ---------- */
.sky {
  position: absolute; inset: 0;
  background:
      radial-gradient(1200px 800px at 70% 20%, rgba(255,255,255,.08), transparent 60%),
      radial-gradient(800px 600px at 20% 10%, rgba(255,255,255,.05), transparent 60%),
      linear-gradient(var(--bg-1), var(--bg-2) 60%, var(--bg-1));
  z-index: -2;
}
.moon {
  position: absolute;
  top: clamp(-64px,-6vw,-24px);
  right: clamp(-24px,-4vw,0px);
  width: min(34vmin, 320px);
  aspect-ratio: 1;
  border-radius: 50%;
  background:
      radial-gradient(circle at 35% 35%, rgba(255,255,255,.35), transparent 45%),
      radial-gradient(circle at 60% 60%, rgba(255,255,255,.2), transparent 46%),
      radial-gradient(circle at 50% 50%, #ffeab5, #ffd36b 60%, #f3b94f 75%, #e6a23a 100%);
  box-shadow:
      0 0 60px 20px rgba(255, 214, 132, .25),
      0 0 160px 40px rgba(255, 214, 132, .12);
  z-index: -1;
}
.shadow { position: absolute; border-radius: 50%; opacity: .35; filter: blur(.4px); background: radial-gradient(circle at 35% 35%, rgba(0,0,0,.25), rgba(0,0,0,.12) 40%, transparent 60%); }
.s1 { width: 16%; aspect-ratio: 1; left: 22%; top: 30%; }
.s2 { width: 10%; aspect-ratio: 1; left: 55%; top: 52%; }
.s3 { width: 13%; aspect-ratio: 1; left: 45%; top: 24%; }

/* ---------- lanterns ---------- */
.lanterns { position: absolute; inset: 0; pointer-events: none; }
.lantern { position: absolute; bottom: -120px; transform: translateY(0) scale(var(--scale,1)); animation: rise var(--dur,16s) linear var(--delay,0s) infinite; }
@keyframes rise {
  0% { transform: translateY(0) scale(var(--scale,1)); opacity: 0; }
  6% { opacity: .9; }
  90% { opacity: .92; }
  100% { transform: translateY(-115vh) scale(var(--scale,1)); opacity: 0; }
}
.lantern-svg { width: clamp(26px, 5vw, 56px); height: auto; filter: drop-shadow(0 6px 10px rgba(0,0,0,.45)); fill: hsl(var(--hue, 32) 85% 60%); }
.lantern-glow { fill: rgba(255, 230, 150, .7); filter: blur(1px); }
.lantern-knot { fill: #3b2b1a; opacity: .8; }

/* ---------- card ---------- */
.container { width: min(780px, 96%); }
.card {
  padding: clamp(16px, 3.2vw, 28px);
  backdrop-filter: blur(8px);
  background: linear-gradient(180deg, rgba(255,255,255,.10), rgba(255,255,255,.06));
  border: 1px solid var(--border);
  border-radius: 18px;
  box-shadow: 0 10px 40px rgba(0,0,0,.35);
  color: var(--text);
}
.stack > * + * { margin-top: .4rem; }
.center { text-align: center; }
.title { margin: 0; font-size: clamp(1.5rem, 3.6vw, 2.2rem); font-weight: 800; text-shadow: 0 2px 18px rgba(255,255,255,.12); }
.subtitle { margin: 0; color: var(--muted); opacity: .95; }
.date { margin: .2rem 0 0; color: #dfe5ff; }

/* ---------- aligned icon rail ---------- */
.icon-rail {
  margin: 14px auto 8px;
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  gap: 10px;
  align-items: center;
}
.icon-btn {
  display: grid;
  grid-template-rows: auto 1fr;
  align-items: center;
  justify-items: center;
  gap: 6px;
  padding: 10px 8px;
  border-radius: 12px;
  background: rgba(255,255,255,.06);
  border: 1px solid var(--border);
  color: var(--text);
  cursor: pointer;
  transition: transform .15s ease, background .15s ease, border-color .15s ease;
}
.icon-btn:hover { transform: translateY(-1px); background: rgba(255,255,255,.10); }
.icon-btn:active { transform: translateY(0); }
.icon-btn[aria-pressed="true"] { border-color: var(--accent); box-shadow: 0 0 0 2px color-mix(in oklab, var(--accent) 40%, transparent); }
.icon-btn span { font-size: .82rem; color: var(--muted); }
.ico { width: 28px; height: 28px; fill: var(--accent); stroke: none; }
.ico-cut { fill: none; stroke: var(--text); stroke-width: 2.5; stroke-linecap: round; stroke-linejoin: round; opacity: .9; }

/* ---------- form & list ---------- */
.wish-form { display: grid; grid-template-columns: 1fr auto; gap: 10px; margin-top: 10px; }
.input {
  height: 44px; padding: 0 14px; border-radius: 12px; outline: none;
  background: rgba(12,16,40,.55); border: 1px solid var(--border); color: var(--text);
}
.input::placeholder { color: color-mix(in oklab, var(--muted) 70%, #fff 30%); }
.btn {
  height: 44px; padding: 0 16px; border-radius: 12px; border: none; cursor: pointer;
  font-weight: 800; color: #1a1a1a; background: linear-gradient(180deg, var(--accent), var(--accent-2));
  box-shadow: 0 6px 18px color-mix(in oklab, var(--accent) 35%, transparent);
}
.btn:disabled { opacity: .55; cursor: not-allowed; }

.list { list-style: none; margin: 12px 0 0; padding: 0; display: grid; gap: 8px; }
.item {
  display: grid; grid-template-columns: 1fr auto; gap: 8px; align-items: center;
  padding: 10px 12px; background: var(--card); border: 1px solid var(--border); border-radius: 12px;
}
.txt { color: #f6f8ff; }
.row { display: inline-grid; grid-auto-flow: column; gap: 8px; align-items: center; }
.ts { font-size: .8rem; color: var(--muted); }
.x {
  width: 28px; height: 28px; border-radius: 8px; border: 1px solid var(--border);
  background: rgba(0,0,0,.25); color: var(--text); cursor: pointer; font-size: 16px;
}

/* ---------- actions ---------- */
.actions { display: flex; flex-wrap: wrap; gap: 8px; justify-content: center; margin-top: 14px; }
.ghost {
  padding: 10px 14px; border-radius: 12px; cursor: pointer;
  border: 1px solid var(--border); background: rgba(255,255,255,.06); color: var(--text);
}
.check { display: inline-flex; align-items: center; gap: 6px; color: var(--muted); }

/* ---------- density toggle ---------- */
:where(.list, .wish-form, .icon-rail) { transition: all .12s ease; }
:where(.page) :where(.list).dense .item,
:where(.page) :where(.list) .item:has(+ .item:empty) { margin: 0; }
:global(.dense) {}
:where(.list) { margin-top: v-bind(dense ? '6px' : '12px'); }
:where(.item) { padding: v-bind(dense ? '8px 10px' : '10px 12px'); }

/* ---------- footer ---------- */
.credit { margin-top: 12px; text-align: center; font-size: 12px; color: color-mix(in oklab, var(--muted) 85%, #fff 15%); }

/* ---------- reduced motion ---------- */
@media (prefers-reduced-motion: reduce) {
  .lantern { animation: none; }
  .icon-btn { transition: none; }
}
</style>