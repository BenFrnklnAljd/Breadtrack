<template>
  <div class="shell">

    <!-- Top Bar -->
    <header class="topbar">
      <div class="topbar-brand">
        <span class="topbar-icon"></span>
        <span class="topbar-name">BreadTrack</span>
      </div>
      <div class="topbar-right">
        <span class="topbar-user">{{ auth.currentUser?.name }}</span>
        <!-- Theme Toggle -->
        <button class="theme-toggle" @click="toggleTheme" :title="isDark ? 'Switch to Light' : 'Switch to Dark'">
          {{ isDark ? '☀️' : '🌙' }}
        </button>
        <button class="topbar-signout" @click="handleLogout">Sign out</button>
      </div>
    </header>

    <!-- Expiry Alerts -->
    <TransitionGroup name="fade">
      <div v-for="alert in visibleAlerts" :key="alert.productId" class="alert-banner">
        <span class="alert-icon">⚠️</span>
        <div class="alert-body">
          <div class="alert-title">{{ alert.name }} — Expired</div>
          <div class="alert-desc">{{ alert.units }} unit{{ alert.units !== 1 ? 's' : '' }} · {{ alert.daysOld }}+ days old</div>
        </div>
        <button class="alert-close" @click="dismissAlert(alert.productId)">✕</button>
      </div>
    </TransitionGroup>

    <!-- Content -->
    <main class="shell-content">
      <RouterView v-slot="{ Component }">
        <Transition name="fade" mode="out-in">
          <component :is="Component" />
        </Transition>
      </RouterView>
    </main>

    <!-- More Drawer -->
    <Transition name="drawer-up">
      <div v-if="showMore" class="overlay" @click="showMore = false">
        <div class="drawer" @click.stop>
          <div class="drawer-handle"></div>
          <div class="drawer-label">More</div>
          <div class="drawer-grid">
            <RouterLink
              v-for="item in moreItems"
              :key="item.to"
              :to="item.to"
              class="drawer-tile"
              active-class="active"
              @click="showMore = false"
            >
              <span class="drawer-tile-icon">{{ item.icon }}</span>
              <span class="drawer-tile-name">{{ item.label }}</span>
            </RouterLink>
          </div>
        </div>
      </div>
    </Transition>

    <!-- Bottom Nav -->
    <nav class="bottomnav">
      <RouterLink
        v-for="item in navItems"
        :key="item.to"
        :to="item.to"
        class="nav-item"
        active-class="active"
      >
        <span class="nav-dot"></span>
        <span class="nav-icon">{{ item.icon }}</span>
        <span class="nav-label">{{ item.label }}</span>
      </RouterLink>
      <button class="nav-item" :class="{ active: showMore }" @click="showMore = !showMore">
        <span class="nav-dot"></span>
        <span class="nav-icon nav-dots">
          <span></span><span></span><span></span>
        </span>
        <span class="nav-label">More</span>
      </button>
    </nav>

  </div>
</template>

<script setup>
import { ref, computed, watch, onMounted } from 'vue'
import { RouterView, RouterLink, useRouter } from 'vue-router'
import { useAuthStore } from '@/stores/auth'
import { useInventoryStore } from '@/stores/inventory'
import { useToastStore } from '@/stores/toast'

const auth   = useAuthStore()
const inv    = useInventoryStore()
const toast  = useToastStore()
const router = useRouter()

const showMore  = ref(false)
const dismissed = ref(new Set())

// ── Theme ──
const isDark = ref(false)

function applyTheme(dark) {
  document.documentElement.setAttribute('data-theme', dark ? 'dark' : 'light')
  localStorage.setItem('theme', dark ? 'dark' : 'light')
}

function toggleTheme() {
  isDark.value = !isDark.value
  applyTheme(isDark.value)
}

onMounted(() => {
  const saved = localStorage.getItem('theme')
  if (saved) {
    isDark.value = saved === 'dark'
  } else {
    isDark.value = window.matchMedia('(prefers-color-scheme: dark)').matches
  }
  applyTheme(isDark.value)
})

// ── Alerts ──
const visibleAlerts = computed(() =>
  inv.expiryAlerts.filter(a => !dismissed.value.has(a.productId))
)

function dismissAlert(pid) {
  dismissed.value = new Set([...dismissed.value, pid])
}

watch(() => inv.expiryAlerts.map(a => a.productId).join(), newIds => {
  const active = new Set(newIds.split(',').filter(Boolean))
  dismissed.value = new Set([...dismissed.value].filter(id => active.has(id)))
})

const navItems = [
  { to: '/dashboard', icon: '⌂',  label: 'Home'    },
  { to: '/produce',   icon: '⊕',  label: 'Produce' },
  { to: '/sales',     icon: '◎',  label: 'Sales'   },
  { to: '/inventory', icon: '▦',  label: 'Stock'   },
  { to: '/reports',   icon: '∿',  label: 'Reports' },
]
const moreItems = [
  { to: '/particulars', icon: '🏷️', label: 'Products' },
  { to: '/expired',     icon: '🗑️', label: 'Expired'  },
]

function handleLogout() {
  auth.logout()
  toast.info('Signed out.')
  router.push('/login')
}
</script>

<style scoped>
.shell {
  display: flex;
  flex-direction: column;
  min-height: 100dvh;
  max-width: var(--max-w);
  margin: 0 auto;
}

/* ── Topbar ── */
.topbar {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 0 16px;
  height: var(--top-h);
  background: var(--bg);
  border-bottom: 1px solid var(--border);
  position: sticky;
  top: 0;
  z-index: 100;
  flex-shrink: 0;
}
.topbar-brand  { display: flex; align-items: center; gap: 8px; }
.topbar-icon   { font-size: 20px; }
.topbar-name   { font-family: var(--font-head); font-size: 16px; font-weight: 800; color: var(--accent); letter-spacing: -.01em; }
.topbar-right  { display: flex; align-items: center; gap: 12px; }
.topbar-user   { font-size: 12px; color: var(--text2); background: var(--surface2); border: 1px solid var(--border); padding: 4px 10px; border-radius: 20px; }
.topbar-signout { font-size: 11px; color: var(--muted); background: none; border: none; cursor: pointer; font-family: var(--font-body); text-transform: uppercase; letter-spacing: .05em; transition: color .15s; }
.topbar-signout:hover { color: var(--red); }

.theme-toggle {
  background: var(--surface2);
  border: 1px solid var(--border);
  border-radius: 20px;
  padding: 4px 8px;
  font-size: 14px;
  cursor: pointer;
  transition: background .2s;
  line-height: 1;
}
.theme-toggle:hover { background: var(--surface3); }

/* ── Content ── */
.shell-content { flex: 1; overflow-y: auto; }

/* ── Bottomnav ── */
.bottomnav {
  display: flex;
  background: var(--nav-bg);
  backdrop-filter: blur(20px);
  -webkit-backdrop-filter: blur(20px);
  border-top: 1px solid var(--border);
  position: sticky;
  bottom: 0;
  z-index: 100;
  flex-shrink: 0;
  padding-bottom: env(safe-area-inset-bottom, 0px);
}
.nav-item {
  flex: 1;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  padding: 10px 4px 12px;
  gap: 4px;
  color: var(--muted);
  text-decoration: none;
  background: none;
  border: none;
  font-family: var(--font-body);
  cursor: pointer;
  position: relative;
  transition: color .2s;
}
.nav-item.active { color: var(--accent); }

.nav-dot {
  position: absolute;
  top: 0; left: 50%;
  transform: translateX(-50%) scaleX(0);
  width: 18px; height: 2px;
  background: var(--accent);
  border-radius: 0 0 2px 2px;
  transition: transform .25s cubic-bezier(.34,1.56,.64,1);
}
.nav-item.active .nav-dot { transform: translateX(-50%) scaleX(1); }

.nav-icon { font-size: 19px; line-height: 1; transition: transform .2s cubic-bezier(.34,1.56,.64,1); }
.nav-item.active .nav-icon { transform: translateY(-1px); }

.nav-dots { display: flex; align-items: center; gap: 3px; height: 19px; font-size: 0; }
.nav-dots span { display: block; width: 4px; height: 4px; border-radius: 50%; background: currentColor; transition: transform .2s; }
.nav-item.active .nav-dots span:nth-child(2) { transform: translateY(-2px); }

.nav-label { font-size: 9px; font-weight: 500; letter-spacing: .06em; text-transform: uppercase; line-height: 1; }

/* ── Drawer ── */
.overlay { position: fixed; inset: 0; background: rgba(0,0,0,.55); backdrop-filter: blur(4px); z-index: 150; display: flex; align-items: flex-end; justify-content: center; }
.drawer { background: var(--surface); border-radius: 16px 16px 0 0; border: 1px solid var(--border); border-bottom: none; width: 100%; max-width: var(--max-w); padding-bottom: calc(var(--nav-h) + 12px); }
.drawer-handle { width: 36px; height: 4px; background: var(--border2); border-radius: 2px; margin: 12px auto 14px; }
.drawer-label { font-size: 10px; font-weight: 700; color: var(--muted); text-transform: uppercase; letter-spacing: .12em; padding: 0 16px 12px; }
.drawer-grid { display: grid; grid-template-columns: 1fr 1fr; gap: 8px; padding: 0 16px; }
.drawer-tile { display: flex; flex-direction: column; align-items: center; justify-content: center; gap: 8px; padding: 20px 12px; background: var(--surface2); border: 1px solid var(--border); border-radius: var(--radius); color: var(--text2); transition: all .15s; }
.drawer-tile.active, .drawer-tile:active { background: var(--accent-dim); border-color: var(--accent); color: var(--accent); }
.drawer-tile-icon { font-size: 26px; }
.drawer-tile-name { font-family: var(--font-head); font-size: 13px; font-weight: 700; }

/* ── Transitions ── */
.drawer-up-enter-active, .drawer-up-leave-active { transition: all .28s cubic-bezier(.4,0,.2,1); }
.drawer-up-enter-from, .drawer-up-leave-to { opacity: 0; }
.drawer-up-enter-from .drawer, .drawer-up-leave-to .drawer { transform: translateY(100%); }
</style>
