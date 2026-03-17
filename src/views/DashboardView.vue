<template>
  <div class="page">
    <div class="page-title">Today's Summary</div>
    <div class="page-subtitle">{{ todayLabel() }}</div>

    <!-- Stats Grid -->
    <div class="stats-grid">
      <StatCard label="Sales Today"  :value="'₱' + inv.todayRevenue.toFixed(2)"  color="text-accent" />
      <StatCard label="Units Sold"   :value="inv.todayUnitsSold"                               />
      <StatCard label="Net Profit"   :value="'₱' + inv.todayProfit.toFixed(2)"   :color="inv.todayProfit >= 0 ? 'text-green' : 'text-red'" />
      <StatCard label="Expiry Alerts" :value="inv.expiryAlerts.length"            color="text-red" />
    </div>

    <!-- Expired Loss Summary -->
    <div v-if="inv.totalExpiredUnits > 0" class="loss-card">
      <div class="loss-row">
        <span class="loss-label">Expired Units</span>
        <span class="loss-val text-red">{{ inv.totalExpiredUnits }}</span>
      </div>
      <div class="loss-row">
        <span class="loss-label">Total Loss Value</span>
        <span class="loss-val text-red">₱{{ inv.totalLossValue.toFixed(2) }}</span>
      </div>
    </div>

    <!-- Current Stock -->
    <div class="section-heading" style="margin-top:8px">Current Stock</div>

    <div v-if="inv.activeProducts.length === 0" class="empty-state">
      <div class="empty-icon">📦</div>
      <div class="empty-text">No products yet.<br/>Go to Products to add some.</div>
    </div>

    <div
      v-for="p in inv.activeProducts"
      :key="p.id"
      class="stock-item"
      :class="stockCardClass(p)"
    >
      <div class="stock-left">
        <div class="stock-name">{{ p.name }}</div>
        <div class="stock-meta">
          <span :class="p.category === 'food' ? 'tag tag-food' : 'tag tag-bev'">{{ p.category }}</span>
          <span class="price-tag">₱{{ p.price }}</span>
        </div>
      </div>
      <div class="stock-right">
        <div class="stock-bal-label">Balance</div>
        <div class="stock-bal" :class="stockBalColor(p)">
          {{ inv.currentBalance(p.id) }}
        </div>
        <span v-if="stockStatus(p) === 'expired'" class="tag tag-expired" style="font-size:10px;margin-top:4px">Pullout</span>
        <span v-else-if="stockStatus(p) === 'restocked'" class="tag tag-ok" style="font-size:10px;margin-top:4px">Restocked</span>
        <span v-else-if="stockStatus(p) === 'empty'" class="tag" style="font-size:10px;margin-top:4px;background:var(--surface2);color:var(--muted)">Empty</span>
      </div>
    </div>
  </div>
</template>

<script setup>
import { useInventoryStore } from '@/stores/inventory'
import { todayLabel } from '@/utils/date'
import StatCard from '@/components/ui/StatCard.vue'

const inv = useInventoryStore()

// Stock status logic:
// - 'expired'   → has expired units AND no new fresh balance (all stock is old)
// - 'restocked' → had expired units but NEW production brought fresh balance back
// - 'ok'        → has balance, nothing expired
// - 'empty'     → zero balance, nothing expired
function stockStatus(p) {
  const balance  = inv.currentBalance(p.id)
  const hasExpired = inv.isExpired(p.id)

  if (hasExpired && balance > 0) return 'restocked'  // new stock added after expiry
  if (hasExpired && balance === 0) return 'expired'   // fully expired, nothing new
  if (balance > 0) return 'ok'
  return 'empty'
}

function stockCardClass(p) {
  const s = stockStatus(p)
  if (s === 'expired')   return 'stock-expired'
  if (s === 'restocked') return 'stock-restocked'
  return ''
}

function stockBalColor(p) {
  const s = stockStatus(p)
  if (s === 'expired')   return 'text-red'
  if (s === 'restocked') return 'text-green'
  if (s === 'ok')        return 'text-green'
  return 'text-muted'
}
</script>

<style scoped>
.loss-card {
  background: var(--red-dim);
  border: 1px solid rgba(224,82,82,.25);
  border-radius: var(--radius-sm);
  padding: 12px 16px;
  margin-bottom: 16px;
}
.loss-row { display: flex; justify-content: space-between; align-items: center; padding: 3px 0; }
.loss-label { font-size: 13px; color: var(--text2); }
.loss-val { font-family: var(--font-head); font-weight: 700; font-size: 16px; }

.stock-item {
  background: var(--surface);
  border: 1px solid var(--border);
  border-radius: var(--radius);
  padding: 14px 16px;
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
  margin-bottom: 10px;
  transition: border-color .2s;
}
.stock-item.stock-expired    { border-color: rgba(224,82,82,.3);  background: rgba(224,82,82,.04); }
.stock-item.stock-restocked { border-color: rgba(76,175,121,.3); background: rgba(76,175,121,.04); transition: all .4s ease; }
.stock-name { font-weight: 600; font-size: 15px; margin-bottom: 6px; }
.stock-meta { display: flex; align-items: center; gap: 8px; }
.stock-right { text-align: right; display: flex; flex-direction: column; align-items: flex-end; }
.stock-bal-label { font-size: 10px; color: var(--muted); text-transform: uppercase; letter-spacing: .06em; }
.stock-bal { font-family: var(--font-head); font-size: 26px; font-weight: 800; line-height: 1; }
</style>