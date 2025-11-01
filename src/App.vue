<template>
  <main class="scene">
    <!-- Night gradient sky -->
    <div class="sky"></div>

    <!-- Big Moon -->
    <div class="moon" aria-hidden="true">
      <div class="crater c1"></div>
      <div class="crater c2"></div>
      <div class="crater c3"></div>
    </div>

    <!-- Floating lanterns -->
    <div class="lanterns">
      <div
          v-for="lan in lanterns"
          :key="lan.id"
          class="lantern"
          :style="lan.style"
          aria-hidden="true"
      >
        <svg viewBox="0 0 64 96" class="lantern-svg">
          <defs>
            <linearGradient id="g" x1="0" y1="0" x2="0" y2="1">
              <stop offset="0%" stop-opacity=".95"/>
              <stop offset="100%" stop-opacity=".65"/>
            </linearGradient>
          </defs>
          <path
              d="M32 2 C50 6 62 20 62 40 v30 c0 18-14 24-30 24S2 88 2 70V40C2 20 14 6 32 2z"
              fill="url(#g)"
          />
          <ellipse cx="32" cy="74" rx="14" ry="8" class="lantern-glow"/>
          <rect x="26" y="86" width="12" height="6" rx="2" class="lantern-knot"/>
        </svg>
      </div>
    </div>

    <!-- Content Card -->
    <section class="card">
      <header class="title-wrap">
        <h1 class="title">한가위 잘 보내세요 🌕</h1>
        <p class="subtitle">
          넉넉한 보름달처럼 마음도 가득 차오르길.
        </p>
        <p class="date" v-if="dText">
          오늘은 <strong>{{ dText }}</strong> 입니다.
        </p>
      </header>

      <form class="wish-form" @submit.prevent="addWish">
        <input
            v-model.trim="draft"
            type="text"
            class="wish-input"
            placeholder="소원을 적어보세요… (Enter)"
            maxlength="80"
        />
        <button class="wish-btn" :disabled="!draft">적기</button>
      </form>

      <ul v-if="wishes.length" class="wish-list">
        <li v-for="w in wishes" :key="w.id" class="wish-item">
          <span class="wish-text">“{{ w.text }}”</span>
          <button class="del-btn" @click="removeWish(w.id)" title="삭제">×</button>
        </li>
      </ul>

      <footer class="actions">
        <button class="ghost" @click="shareOrCopy">페이지 공유하기</button>
        <button class="ghost" @click="clearAll" :disabled="!wishes.length">전체 삭제</button>
      </footer>
    </section>

    <footer class="credit">
      <span>© {{ year }} Chuseok • Vue + TS + Vite</span>
    </footer>
  </main>
</template>

<script setup lang="ts">
import { computed, onMounted, reactive, ref, watch } from 'vue'

type Wish = { id: string; text: string }
const LS_KEY = 'chuseok:wishes'

const wishes = ref<Wish[]>([])
const draft = ref<string>('')

/** Load & persist wishes */
onMounted(() => {
  try {
    const raw = localStorage.getItem(LS_KEY)
    if (raw) wishes.value = JSON.parse(raw) as Wish[]
  } catch { /* noop */ }
})
watch(wishes, v => {
  localStorage.setItem(LS_KEY, JSON.stringify(v))
}, { deep: true })

function addWish() {
  if (!draft.value) return
  wishes.value.unshift({ id: crypto.randomUUID(), text: draft.value })
  draft.value = ''
}
function removeWish(id: string) {
  wishes.value = wishes.value.filter(w => w.id !== id)
}
function clearAll() {
  if (!wishes.value.length) return
  if (confirm('모든 소원을 삭제할까요?')) wishes.value = []
}

/** Share / Copy */
async function shareOrCopy() {
  const text = '한가위 복 많이 받으세요 🌕'
  const url = location.href
  try {
    if ((navigator as any).share) {
      await (navigator as any).share({ title: document.title || text, text, url })
    } else {
      await navigator.clipboard.writeText(url)
      alert('링크를 클립보드에 복사했어요!')
    }
  } catch {
    /* user canceled */
  }
}

/** Pretty date text (KST) */
const dText = computed(() => {
  try {
    const now = new Date()
    const opts: Intl.DateTimeFormatOptions = {
      year: 'numeric', month: 'long', day: 'numeric',
      weekday: 'long'
    }
    return new Intl.DateTimeFormat('ko-KR', opts).format(now)
  } catch {
    return ''
  }
})

/** Lanterns (animated) */
type Lantern = { id: number; style: Record<string, string> }
const lanterns = reactive<Lantern[]>([])
function rand(min: number, max: number) {
  return Math.random() * (max - min) + min
}
function spawnLanterns(n = 10) {
  for (let i = 0; i < n; i++) {
    const left = `${rand(0, 100)}%`
    const delay = `${rand(0, 6).toFixed(2)}s`
    const duration = `${rand(12, 22).toFixed(1)}s`
    const scale = rand(0.8, 1.2).toFixed(2)
    const hue = Math.floor(rand(18, 48)) // warm
    lanterns.push({
      id: i,
      style: {
        left,
        '--delay': delay,
        '--duration': duration,
        '--scale': scale,
        '--hue': String(hue)
      } as Record<string, string>
    })
  }
}
onMounted(() => spawnLanterns(12))

const year = new Date().getFullYear()
</script>

<style scoped>
/* --- Layout --- */
:root { color-scheme: dark; }
.scene {
  position: relative;
  min-height: 100svh;
  overflow: hidden;
  display: grid;
  place-items: center;
  padding: clamp(16px, 2vw, 32px);
  font-family: ui-rounded, system-ui, -apple-system, "Apple SD Gothic Neo",
  "Noto Sans KR", "Segoe UI", Roboto, "Helvetica Neue", Arial, "Malgun Gothic",
  "맑은 고딕", sans-serif;
}
.sky {
  position: absolute; inset: 0;
  background:
      radial-gradient(1200px 800px at 70% 20%, rgba(255,255,255,.07), transparent 60%),
      radial-gradient(800px 600px at 20% 10%, rgba(255,255,255,.05), transparent 60%),
      linear-gradient(#0a0f24, #1a2447 60%, #0b0f24);
  z-index: -2;
}

/* --- Moon --- */
.moon {
  position: absolute;
  top: clamp(-80px, -6vw, -40px);
  right: clamp(-40px, -3vw, -10px);
  width: min(34vmin, 320px);
  aspect-ratio: 1/1;
  border-radius: 50%;
  background:
      radial-gradient(circle at 35% 35%, rgba(255,255,255,.35), transparent 45%),
      radial-gradient(circle at 60% 60%, rgba(255,255,255,.2), transparent 46%),
      radial-gradient(circle at 50% 50%, #ffeab5, #ffd36b 60%, #f3b94f 75%, #e6a23a 100%);
  box-shadow:
      0 0 60px 20px rgba(255, 214, 132, .25),
      0 0 160px 40px rgba(255, 214, 132, .15);
  z-index: -1;
  transform: translate3d(0,0,0);
}
.crater {
  position: absolute;
  background: radial-gradient(circle at 35% 35%, rgba(0,0,0,.25), rgba(0,0,0,.15) 40%, rgba(0,0,0,0) 60%);
  border-radius: 50%;
  opacity: .35;
  filter: blur(0.3px);
}
.c1 { width: 16%; aspect-ratio: 1; left: 22%; top: 30%; }
.c2 { width: 10%; aspect-ratio: 1; left: 55%; top: 52%; }
.c3 { width: 13%; aspect-ratio: 1; left: 45%; top: 24%; }

/* --- Lanterns --- */
.lanterns { position: absolute; inset: 0; pointer-events: none; }
.lantern {
  position: absolute; bottom: -120px;
  animation: rise var(--duration, 16s) linear var(--delay, 0s) infinite;
  transform: translateY(0) scale(var(--scale, 1));
}
@keyframes rise {
  0%   { transform: translateY(0)     scale(var(--scale, 1)); opacity: 0; }
  5%   { opacity: .9; }
  90%  { opacity: .9; }
  100% { transform: translateY(-115vh) scale(var(--scale, 1)); opacity: 0; }
}
.lantern-svg {
  width: clamp(26px, 5vw, 56px);
  height: auto;
  filter: drop-shadow(0 6px 10px rgba(0,0,0,.45));
  fill: hsl(var(--hue, 32) 85% 60%);
}
.lantern-glow {
  fill: rgba(255, 230, 150, .7);
  filter: blur(1px);
}
.lantern-knot { fill: #3b2b1a; opacity: .8; }

/* --- Card --- */
.card {
  width: min(760px, 96%);
  backdrop-filter: blur(8px);
  background: linear-gradient(180deg, rgba(255,255,255,.12), rgba(255,255,255,.06));
  border: 1px solid rgba(255,255,255,.15);
  border-radius: 20px;
  padding: clamp(16px, 3vw, 28px);
  box-shadow: 0 10px 40px rgba(0,0,0,.35);
}
.title-wrap { text-align: center; margin-bottom: 12px; }
.title {
  margin: 0 0 4px;
  font-size: clamp(1.4rem, 3.5vw, 2.2rem);
  letter-spacing: .2px;
  font-weight: 800;
  color: #fff;
  text-shadow: 0 2px 18px rgba(255,255,255,.12);
}
.subtitle {
  margin: 0;
  color: #e7eaff;
  opacity: .9;
  font-size: clamp(.95rem, 2.4vw, 1.05rem);
}
.date {
  margin: 8px 0 0;
  font-size: .95rem;
  color: #d6dbff;
  opacity: .9;
}

/* --- Wish form --- */
.wish-form {
  display: grid;
  grid-template-columns: 1fr auto;
  gap: 10px;
  margin-top: 18px;
}
.wish-input {
  height: 44px;
  border-radius: 12px;
  padding: 0 14px;
  border: 1px solid rgba(255,255,255,.25);
  background: rgba(12,16,40,.55);
  color: #fff;
  outline: none;
}
.wish-input::placeholder { color: rgba(230,235,255,.6); }
.wish-btn {
  height: 44px;
  padding: 0 16px;
  border: none;
  border-radius: 12px;
  font-weight: 700;
  cursor: pointer;
  color: #1a1a1a;
  background: linear-gradient(180deg, #ffd36b, #ffc34b);
  box-shadow: 0 6px 18px rgba(255, 196, 75, .35);
}
.wish-btn:disabled { opacity: .5; cursor: not-allowed; }

/* --- Wish list --- */
.wish-list { list-style: none; padding: 0; margin: 14px 0 0; display: grid; gap: 8px; }
.wish-item {
  display: grid; grid-template-columns: 1fr auto; align-items: center;
  gap: 8px;
  padding: 10px 12px;
  border-radius: 12px;
  background: rgba(255,255,255,.06);
  border: 1px solid rgba(255,255,255,.12);
}
.wish-text { color: #f3f6ff; }
.del-btn {
  width: 28px; height: 28px; border-radius: 8px;
  border: 1px solid rgba(255,255,255,.25);
  background: rgba(0,0,0,.2);
  color: #fff; cursor: pointer; font-size: 16px; line-height: 1;
}

/* --- Actions --- */
.actions { display: flex; gap: 8px; justify-content: center; margin-top: 16px; flex-wrap: wrap; }
.ghost {
  padding: 10px 14px; border-radius: 12px; cursor: pointer; border: 1px solid rgba(255,255,255,.25);
  background: rgba(255,255,255,.06); color: #fff;
}

/* --- Credit --- */
.credit {
  position: absolute; inset-inline: 0; bottom: 10px;
  text-align: center; font-size: 12px; color: rgba(230,235,255,.7);
}
</style>