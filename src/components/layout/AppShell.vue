<template>
  <div class="app-shell">

    <!-- Top Bar -->
    <header class="top-bar">
      <div class="top-bar-left">
        <span class="top-bar-logo">🥐</span>
        <span class="top-bar-title">PerishTrack</span>
      </div>
      <div class="top-bar-right">
        <span class="top-bar-user">{{ auth.currentUser?.name }}</span>
        <button class="logout-btn" @click="handleLogout">Sign out</button>
      </div>
    </header>

    <!-- Expiry Alerts -->
    <TransitionGroup name="fade">
      <div v-for="alert in visibleAlerts" :key="alert.productId" class="alert-banner">
        <div class="alert-icon">⚠️</div>
        <div class="alert-body">
          <div class="alert-title">{{ alert.name }} — Expired</div>
          <div class="alert-desc">{{ alert.units }} unit{{ alert.units !== 1 ? 's' : '' }} · {{ alert.daysOld }}+ days old</div>
        </div>
        <button class="alert-close" @click="dismissAlert(alert.productId)">✕</button>
      </div>
    </TransitionGroup>

    <!-- Page Content -->
    <main class="shell-main">
      <RouterView v-slot="{ Component }">
        <Transition name="fade" mode="out-in">
          <component :is="Component" />
        </Transition>
      </RouterView>
    </main>

    <!-- More Drawer -->
    <Transition name="drawer-up">
      <div v-if="showMore" class="more-overlay" @click="showMore = false">
        <div class="more-drawer" @click.stop>
          <div class="drawer-handle"></div>
          <div class="drawer-heading">Quick Access</div>
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
              <span class="drawer-tile-label">{{ item.label }}</span>
            </RouterLink>
          </div>
        </div>
      </div>
    </Transition>

    <!-- Bottom Nav -->
    <nav class="bottom-nav">
      <RouterLink
        v-for="item in navItems"
        :key="item.to"
        :to="item.to"
        class="nav-item"
        active-class="active"
      >
        <span class="nav-pip"></span>
        <span class="nav-icon">{{ item.icon }}</span>
        <span class="nav-label">{{ item.label }}</span>
      </RouterLink>
      <button
        class="nav-item"
        :class="{ active: showMore }"
        @click="showMore = !showMore"
      >
        <span class="nav-pip"></span>
        <span class="nav-icon nav-more-icon">
          <span></span><span></span><span></span>
        </span>
        <span class="nav-label">More</span>
      </button>
    </nav>

  </div>
</template>

<script setup>
import { ref, computed, watch } from 'vue'
import { RouterView, RouterLink, useRouter } from 'vue-router'
import { useAuthStore } from '@/stores/auth'
import { useInventoryStore } from '@/stores/inventory'
import { useToastStore } from '@/stores/toast'

const auth   = useAuthStore()
const inv    = useInventoryStore()
const toast  = useToastStore()
const router = useRouter()

const dismissed = ref(new Set())
const showMore  = ref(false)

const visibleAlerts = computed(() =>
  inv.expiryAlerts.filter(a => !dismissed.value.has(a.productId))
)

function dismissAlert(productId) {
  dismissed.value = new Set([...dismissed.value, productId])
}

watch(() => inv.expiryAlerts.map(a => a.productId).join(), (newIds) => {
  const activeIds = new Set(newIds.split(',').filter(Boolean))
  dismissed.value = new Set([...dismissed.value].filter(id => activeIds.has(id)))
})

const navItems = [
  { to: '/dashboard', icon: '⌂',  label: 'Home'    },
  { to: '/produce',   icon: '⊕',  label: 'Produce' },
  { to: '/sales',     icon: '◎',  label: 'Sales'   },
  { to: '/inventory', icon: '▦',  label: 'Stock'   },
  { to: '/reports',   icon: '∿',  label: 'Reports' },
]

const moreItems = [
  { to: '/particulars', label: 'Products', icon: '🏷️' },
  { to: '/expired',     label: 'Expired',  icon: '🗑️' },
]

function handleLogout() {
  auth.logout()
  toast.info('Logged out.')
  router.push('/login')
}
</script>

<style scoped>
.app-shell {
  display: flex;
  flex-direction: column;
  min-height: 100dvh;
  max-width: var(--max-w);
  margin: 0 auto;
}

/* ── Top Bar ── */
.top-bar {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 0 20px;
  height: var(--top-h);
  background: var(--bg);
  border-bottom: 1px solid var(--border);
  position: sticky;
  top: 0;
  z-index: 100;
  flex-shrink: 0;
}
.top-bar-left  { display: flex; align-items: center; gap: 8px; }
.top-bar-logo  { font-size: 20px; }
.top-bar-title {
  font-family: var(--font-head);
  font-size: 16px;
  font-weight: 800;
  color: var(--accent);
  letter-spacing: -.01em;
}
.top-bar-right { display: flex; align-items: center; gap: 14px; }
.top-bar-user  {
  font-size: 12px;
  color: var(--muted);
  background: var(--surface2);
  padding: 4px 10px;
  border-radius: 20px;
  border: 1px solid var(--border);
}
.logout-btn {
  font-size: 11px;
  color: var(--muted);
  background: none;
  border: none;
  cursor: pointer;
  font-family: var(--font-body);
  letter-spacing: .04em;
  text-transform: uppercase;
  transition: color .15s;
}
.logout-btn:hover { color: var(--red); }

/* ── Alerts ── */
.alert-banner {
  display: flex;
  align-items: center;
  gap: 10px;
  margin: 8px 16px 0;
  background: rgba(224,82,82,.08);
  border: 1px solid rgba(224,82,82,.2);
  border-radius: var(--radius-sm);
  padding: 10px 12px;
  backdrop-filter: blur(4px);
}
.alert-icon  { font-size: 14px; flex-shrink: 0; }
.alert-body  { flex: 1; }
.alert-title { font-weight: 600; font-size: 12px; color: var(--red); }
.alert-desc  { font-size: 11px; color: var(--muted); margin-top: 1px; }
.alert-close {
  background: none; border: none; color: var(--muted);
  font-size: 12px; cursor: pointer; padding: 4px;
  flex-shrink: 0; line-height: 1; transition: color .15s;
  border-radius: 4px;
}
.alert-close:hover { color: var(--red); background: var(--red-dim); }

/* ── Main ── */
.shell-main { flex: 1; overflow-y: auto; }

/* ── Bottom Nav ── */
.bottom-nav {
  display: flex;
  align-items: stretch;
  background: rgba(15,14,12,.92);
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
  cursor: pointer;
  color: var(--muted);
  text-decoration: none;
  background: none;
  border: none;
  font-family: var(--font-body);
  position: relative;
  transition: color .2s;
  -webkit-tap-highlight-color: transparent;
}
.nav-item.active { color: var(--accent); }

/* Active indicator pip */
.nav-pip {
  position: absolute;
  top: 0;
  left: 50%;
  transform: translateX(-50%) scaleX(0);
  width: 20px;
  height: 2px;
  background: var(--accent);
  border-radius: 0 0 2px 2px;
  transition: transform .25s cubic-bezier(.34,1.56,.64,1);
}
.nav-item.active .nav-pip { transform: translateX(-50%) scaleX(1); }

.nav-icon {
  font-size: 20px;
  line-height: 1;
  transition: transform .2s cubic-bezier(.34,1.56,.64,1);
}
.nav-item.active .nav-icon { transform: translateY(-1px); }

/* Three-dot more icon (CSS only, no emoji) */
.nav-more-icon {
  display: flex;
  align-items: center;
  gap: 3px;
  height: 20px;
  font-size: 0;
}
.nav-more-icon span {
  display: block;
  width: 4px;
  height: 4px;
  border-radius: 50%;
  background: currentColor;
  transition: transform .2s;
}
.nav-item.active .nav-more-icon span:nth-child(2) { transform: translateY(-2px); }

.nav-label {
  font-size: 9px;
  font-weight: 500;
  letter-spacing: .06em;
  text-transform: uppercase;
  line-height: 1;
}

/* ── More Drawer ── */
.more-overlay {
  position: fixed;
  inset: 0;
  background: rgba(0,0,0,.6);
  backdrop-filter: blur(4px);
  z-index: 150;
  display: flex;
  align-items: flex-end;
  justify-content: center;
}
.more-drawer {
  background: var(--surface);
  border-radius: 20px 20px 0 0;
  border: 1px solid var(--border);
  border-bottom: none;
  width: 100%;
  max-width: var(--max-w);
  padding-bottom: calc(var(--nav-h) + 12px);
}
.drawer-handle {
  width: 36px;
  height: 4px;
  background: var(--border2);
  border-radius: 2px;
  margin: 12px auto 16px;
}
.drawer-heading {
  font-family: var(--font-head);
  font-size: 11px;
  font-weight: 700;
  color: var(--muted);
  text-transform: uppercase;
  letter-spacing: .12em;
  padding: 0 20px 12px;
}
.drawer-grid {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 10px;
  padding: 0 16px;
}
.drawer-tile {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  gap: 8px;
  padding: 20px 12px;
  background: var(--surface2);
  border: 1px solid var(--border);
  border-radius: var(--radius);
  text-decoration: none;
  color: var(--text2);
  transition: all .18s;
}
.drawer-tile:active, .drawer-tile.active {
  background: var(--accent-dim);
  border-color: var(--accent);
  color: var(--accent);
}
.drawer-tile-icon  { font-size: 28px; line-height: 1; }
.drawer-tile-label {
  font-family: var(--font-head);
  font-size: 13px;
  font-weight: 700;
  letter-spacing: .02em;
}

/* ── Transitions ── */
.drawer-up-enter-active, .drawer-up-leave-active { transition: all .28s cubic-bezier(.4,0,.2,1); }
.drawer-up-enter-from .more-drawer, .drawer-up-leave-to .more-drawer { transform: translateY(100%); }
.drawer-up-enter-from, .drawer-up-leave-to { opacity: 0; }
</style>