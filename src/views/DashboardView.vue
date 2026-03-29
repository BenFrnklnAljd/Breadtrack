<template>
  <div class="page">
    <div class="dash-header">
      <div class="page-title">Good {{ greeting }}, {{ auth.currentUser?.name }} 👋</div>
      <div class="page-subtitle">{{ todayLabel() }}</div>
    </div>
    <div class="stats-grid">
      <div class="stat-card"><div class="stat-label">Sales Today</div><div class="stat-val val-accent">₱{{ inv.todayRevenue.toFixed(2) }}</div></div>
      <div class="stat-card"><div class="stat-label">Units Sold</div><div class="stat-val">{{ inv.todayUnitsSold }}</div></div>
      <div class="stat-card"><div class="stat-label">Net Profit</div><div class="stat-val" :class="inv.todayProfit >= 0 ? 'val-green' : 'val-red'">₱{{ inv.todayProfit.toFixed(2) }}</div></div>
      <div class="stat-card"><div class="stat-label">Alerts</div><div class="stat-val" :class="inv.expiryAlerts.length > 0 ? 'val-red' : 'val-muted'">{{ inv.expiryAlerts.length }}</div></div>
    </div>
    <div v-if="inv.totalExpiredUnits > 0" class="loss-strip">
      <div class="loss-row"><span>Expired units</span><span class="text-red">{{ inv.totalExpiredUnits }}</span></div>
      <div class="loss-row"><span>Total loss</span><span class="text-red">₱{{ inv.totalLossValue.toFixed(2) }}</span></div>
    </div>
    <div class="section-label">Current Stock</div>
    <div v-if="inv.activeProducts.length === 0" class="empty-state">
      <div class="empty-icon">📦</div>
      <div class="empty-text">No products yet.<br/>Go to Products to add some.</div>
    </div>
    <div v-for="p in inv.activeProducts" :key="p.id" class="stock-row" :class="stockCardClass(p)">
      <div class="stock-left">
        <div class="stock-name">{{ p.name }}</div>
        <div class="stock-meta">
          <span :class="p.category === 'food' ? 'tag tag-food' : 'tag tag-bev'">{{ p.category }}</span>
          <span class="price-tag">₱{{ p.price }}</span>
        </div>
      </div>
      <div class="stock-right">
        <div class="stock-bal-label">Balance</div>
        <div class="stock-bal" :class="stockBalColor(p)">{{ inv.currentBalance(p.id) }}</div>
        <span v-if="stockStatus(p) === 'expired'"    class="tag tag-expired" style="font-size:10px;margin-top:3px">Pullout</span>
        <span v-else-if="stockStatus(p) === 'restocked'" class="tag tag-ok"  style="font-size:10px;margin-top:3px">Restocked</span>
        <span v-else-if="stockStatus(p) === 'empty'"     class="tag tag-muted" style="font-size:10px;margin-top:3px">Empty</span>
      </div>
    </div>
  </div>
</template>
<script setup>
import { computed } from 'vue'
import { useInventoryStore } from '@/stores/inventory'
import { useAuthStore } from '@/stores/auth'
import { todayLabel } from '@/utils/date'
const inv = useInventoryStore(); const auth = useAuthStore()
const greeting = computed(() => { const h = new Date().getHours(); if (h < 12) return 'morning'; if (h < 17) return 'afternoon'; return 'evening' })
function stockStatus(p) { const bal = inv.currentBalance(p.id); const exp = inv.isExpired(p.id); if (exp && bal > 0) return 'restocked'; if (exp && bal === 0) return 'expired'; if (bal > 0) return 'ok'; return 'empty' }
function stockCardClass(p) { const s = stockStatus(p); if (s === 'expired') return 'stock-expired'; if (s === 'restocked') return 'stock-restocked'; return '' }
function stockBalColor(p) { const s = stockStatus(p); if (s === 'expired') return 'text-red'; if (s === 'restocked' || s === 'ok') return 'text-green'; return 'text-muted' }
</script>
<style scoped>
.dash-header { margin-bottom: 20px; }
.loss-strip { background: var(--red-dim); border: 1px solid rgba(224,82,82,.2); border-radius: var(--radius-sm); padding: 12px 14px; margin-bottom: 4px; display: flex; flex-direction: column; gap: 6px; }
.loss-row { display: flex; justify-content: space-between; font-size: 13px; color: var(--text2); }
.loss-row span:last-child { font-family: var(--font-head); font-weight: 700; font-size: 14px; }
.stock-row { display: flex; justify-content: space-between; align-items: center; background: var(--surface); border: 1px solid var(--border); border-radius: var(--radius); padding: 14px; margin-bottom: 8px; transition: border-color .2s, background .2s; }
.stock-row.stock-expired   { border-color: rgba(224,82,82,.25);  background: rgba(224,82,82,.04); }
.stock-row.stock-restocked { border-color: rgba(61,184,122,.25); background: rgba(61,184,122,.04); }
.stock-left { flex: 1; }
.stock-name { font-weight: 600; font-size: 15px; margin-bottom: 6px; }
.stock-meta { display: flex; align-items: center; gap: 8px; }
.stock-right { text-align: right; display: flex; flex-direction: column; align-items: flex-end; gap: 2px; }
.stock-bal-label { font-size: 10px; color: var(--muted); text-transform: uppercase; letter-spacing: .07em; }
.stock-bal { font-family: var(--font-head); font-size: 28px; font-weight: 800; line-height: 1; }
</style>