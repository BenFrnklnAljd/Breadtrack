<template>
  <div class="page">
    <div class="page-title">Stock Overview</div>
    <div class="page-subtitle">Full inventory computation</div>

    <!-- Grand Summary -->
    <div class="summary-section">

      <!-- Section label -->
      

      <!-- 4 Stat Tiles -->
      <div class="stat-tiles">
        <div class="stat-tile">
          <div class="stat-tile-icon">🏭</div>
          <div class="stat-tile-val">{{ grandTotals.produced }}</div>
          <div class="stat-tile-label">Total Produced</div>
          <div class="stat-tile-hint">All batches ever made</div>
        </div>
        <div class="stat-tile green">
          <div class="stat-tile-icon">🛒</div>
          <div class="stat-tile-val text-green">{{ grandTotals.sold }}</div>
          <div class="stat-tile-label">Total Sold</div>
          <div class="stat-tile-hint">Units customers bought</div>
        </div>
        <div class="stat-tile red">
          <div class="stat-tile-icon">🗑️</div>
          <div class="stat-tile-val text-red">{{ grandTotals.expired }}</div>
          <div class="stat-tile-label">Expired / Pulled Out</div>
          <div class="stat-tile-hint">Food items past 2 days</div>
        </div>
        <div class="stat-tile accent">
          <div class="stat-tile-icon">📦</div>
          <div class="stat-tile-val text-accent">{{ grandTotals.remaining }}</div>
          <div class="stat-tile-label">Still In Stock</div>
          <div class="stat-tile-hint">Ready to sell now</div>
        </div>
      </div>

      <!-- Visual Formula -->
      <div class="formula-box">
        <div class="formula-box-title">How remaining stock is calculated:</div>
        <div class="formula-row">
          <div class="formula-item">
            <div class="formula-num">{{ grandTotals.produced }}</div>
            <div class="formula-sub">Produced</div>
          </div>
          <div class="formula-op">−</div>
          <div class="formula-item">
            <div class="formula-num text-green">{{ grandTotals.sold }}</div>
            <div class="formula-sub">Sold</div>
          </div>
          <div class="formula-op">−</div>
          <div class="formula-item">
            <div class="formula-num text-red">{{ grandTotals.expired }}</div>
            <div class="formula-sub">Expired</div>
          </div>
          <div class="formula-op">=</div>
          <div class="formula-item result">
            <div class="formula-num text-accent">{{ grandTotals.remaining }}</div>
            <div class="formula-sub">Remaining</div>
          </div>
        </div>
      </div>

      <!-- Money Summary -->
      <div class="money-card">
        <div class="money-title">Revenue Summary</div>
        <div class="money-row">
          <div class="money-left">
            <span class="money-icon"> </span>
            <div>
              <div class="money-label">Sales Revenue</div>
              <div class="money-hint">Money collected from customers</div>
            </div>
          </div>
          <span class="money-val text-green">₱{{ grandTotals.revenue.toFixed(2) }}</span>
        </div>
        <div class="money-row">
          <div class="money-left">
            <span class="money-icon"> </span>
            <div>
              <div class="money-label">Loss from Expired</div>
              <div class="money-hint">Value of items thrown out</div>
            </div>
          </div>
          <span class="money-val text-red">−₱{{ grandTotals.loss.toFixed(2) }}</span>
        </div>
        <div class="money-divider"></div>
        <div class="money-result" :class="grandTotals.net >= 0 ? 'profit' : 'loss'">
          <div class="money-result-left">
            <span class="money-result-icon">{{ grandTotals.net >= 0 ? ' ' : '📉' }}</span>
            <div>
              <div class="money-result-label">{{ grandTotals.net >= 0 ? 'Net Profit' : 'Net Loss' }}</div>
              <div class="money-result-hint">{{ grandTotals.net >= 0 ? 'Revenue minus expired losses' : 'Losses exceed revenue' }}</div>
            </div>
          </div>
          <span class="money-result-val">₱{{ grandTotals.net.toFixed(2) }}</span>
        </div>
      </div>

    </div>

    <!-- Filter -->
    <div class="form-group" style="margin-top:4px">
      <label class="field-label">Filter by Category</label>
      <select class="field-input" v-model="filterCat">
        <option value="">All Categories</option>
        <option value="food">🍞 Food</option>
        <option value="beverage">☕ Beverage</option>
      </select>
    </div>

    <!-- Per-Product Table -->
    <div class="section-heading">Per Product Breakdown</div>

    <div v-if="filteredProducts.length === 0" class="empty-state">
      <div class="empty-icon">📋</div>
      <div class="empty-text">No active products found.</div>
    </div>

    <div v-else>
      <div class="inv-thead">
        <span>Product</span>
        <span class="col-c">Prod.</span>
        <span class="col-c">Sold</span>
        <span class="col-c">Exp.</span>
        <span class="col-c">Bal.</span>
      </div>
      <div
        v-for="p in filteredProducts"
        :key="p.id"
        :class="['inv-row', inv.isExpired(p.id) ? 'expired' : '']"
        @click="selectProduct(p)"
      >
        <div class="inv-cell">
          <div>
            <div class="inv-name">{{ p.name }}</div>
            <span :class="p.category === 'food' ? 'tag tag-food' : 'tag tag-bev'" style="font-size:10px;margin-top:3px;display:inline-flex">
              {{ p.category }}
            </span>
          </div>
        </div>
        <div class="inv-cell col-c">{{ inv.totalProduced(p.id) }}</div>
        <div class="inv-cell col-c text-green">{{ inv.totalSold(p.id) }}</div>
        <div class="inv-cell col-c text-red">{{ inv.expiredUnits(p.id) }}</div>
        <div class="inv-cell col-c">
          <span
            class="inv-num"
            :class="inv.isExpired(p.id) ? 'text-red' : inv.currentBalance(p.id) > 0 ? 'text-green' : 'text-muted'"
          >
            {{ inv.currentBalance(p.id) }}
          </span>
        </div>
      </div>
    </div>

    <!-- Product Detail Modal -->
    <BaseModal v-model="showDetail" :title="selected?.name">
      <div v-if="selected">
        <div style="display:flex;gap:8px;margin-bottom:16px;flex-wrap:wrap">
          <span :class="selected.category === 'food' ? 'tag tag-food' : 'tag tag-bev'">{{ selected.category }}</span>
          <span class="price-tag">₱{{ selected.price }} / unit</span>
          <span v-if="inv.isExpired(selected.id)" class="tag tag-expired">Has Expired Stock</span>
          <span v-else class="tag tag-ok">OK</span>
        </div>

        <div class="detail-section-title">📊 Computation</div>
        <div class="detail-grid">
          <div class="detail-row">
            <span class="detail-label">Total Produced</span>
            <span class="detail-val">{{ inv.totalProduced(selected.id) }} units</span>
          </div>
          <div class="detail-row">
            <span class="detail-label">Total Sold</span>
            <span class="detail-val text-green">{{ inv.totalSold(selected.id) }} units</span>
          </div>
          <div class="detail-row">
            <span class="detail-label">Expired / Pullout</span>
            <span class="detail-val text-red">{{ inv.expiredUnits(selected.id) }} units</span>
          </div>
          <div class="detail-row formula-row">
            <span class="detail-label">Formula</span>
            <span class="formula-text">
              {{ inv.totalProduced(selected.id) }} − {{ inv.totalSold(selected.id) }} − {{ inv.expiredUnits(selected.id) }} = {{ inv.currentBalance(selected.id) }}
            </span>
          </div>
          <div class="detail-row highlight-row">
            <span class="detail-label" style="font-weight:600">Remaining Balance</span>
            <span class="detail-val" style="font-size:22px" :class="inv.currentBalance(selected.id) > 0 ? 'text-green' : 'text-muted'">
              {{ inv.currentBalance(selected.id) }} units
            </span>
          </div>
        </div>

        <div class="detail-section-title" style="margin-top:14px">💰 Financial</div>
        <div class="detail-grid">
          <div class="detail-row">
            <span class="detail-label">Revenue Generated</span>
            <span class="detail-val text-green">₱{{ productRevenue(selected.id).toFixed(2) }}</span>
          </div>
          <div class="detail-row">
            <span class="detail-label">Loss (Expired)</span>
            <span class="detail-val text-red">₱{{ (inv.expiredUnits(selected.id) * selected.price).toFixed(2) }}</span>
          </div>
          <div class="detail-row highlight-row">
            <span class="detail-label" style="font-weight:600">Net Profit</span>
            <span
              class="detail-val"
              style="font-size:18px"
              :class="netProfit(selected) >= 0 ? 'text-green' : 'text-red'"
            >
              ₱{{ netProfit(selected).toFixed(2) }}
            </span>
          </div>
        </div>

        <div class="detail-section-title" style="margin-top:14px">🏭 Recent Batches</div>
        <div v-for="batch in recentBatches(selected.id)" :key="batch.id" class="batch-row">
          <div>
            <div style="font-size:13px;font-weight:500">{{ batch.qty }} units</div>
            <div class="date-meta">{{ batch.date }} · {{ batch.staffName }}</div>
          </div>
          <span v-if="selected.category === 'food'" :class="['tag', expiryTagClass(batch.date)]" style="font-size:10px">
            {{ expiryLabel(batch.date) }}
          </span>
          <span v-else class="tag tag-bev" style="font-size:10px">No Expiry</span>
        </div>
        <div v-if="recentBatches(selected.id).length === 0" style="font-size:13px;color:var(--muted)">No production entries.</div>

        <button class="btn btn-ghost btn-block" style="margin-top:16px" @click="showDetail = false">Close</button>
      </div>
    </BaseModal>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'
import { useInventoryStore } from '@/stores/inventory'
import { expiryLabel, expiryTagClass } from '@/utils/date'
import BaseModal from '@/components/ui/BaseModal.vue'

const inv = useInventoryStore()

const filterCat  = ref('')
const showDetail = ref(false)
const selected   = ref(null)

const filteredProducts = computed(() =>
  inv.activeProducts.filter(p => !filterCat.value || p.category === filterCat.value)
)

const grandTotals = computed(() => {
  const produced  = inv.activeProducts.reduce((s, p) => s + inv.totalProduced(p.id), 0)
  const sold      = inv.activeProducts.reduce((s, p) => s + inv.totalSold(p.id), 0)
  const expired   = inv.activeProducts.reduce((s, p) => s + inv.expiredUnits(p.id), 0)
  const remaining = inv.activeProducts.reduce((s, p) => s + inv.currentBalance(p.id), 0)
  const revenue   = inv.sales.reduce((s, x) => s + x.qty * x.price, 0)
  const loss      = inv.totalLossValue
  return { produced, sold, expired, remaining, revenue, loss, net: revenue - loss }
})

function selectProduct(p) {
  selected.value = p
  showDetail.value = true
}

function productRevenue(pid) {
  return inv.sales.filter(s => s.productId === pid).reduce((s, x) => s + x.qty * x.price, 0)
}

function netProfit(p) {
  return productRevenue(p.id) - inv.expiredUnits(p.id) * p.price
}

function recentBatches(pid) {
  return inv.productions
    .filter(p => p.productId === pid)
    .sort((a, b) => b.date.localeCompare(a.date))
    .slice(0, 5)
}
</script>

<style scoped>
/* ── Summary Section ── */
.summary-section { margin-bottom: 20px; }
.summary-section-label {
  font-family: var(--font-head);
  font-size: 16px;
  font-weight: 800;
  margin-bottom: 4px;
}
.summary-section-hint {
  font-size: 12px;
  color: var(--muted);
  margin-bottom: 14px;
}

/* Stat Tiles */
.stat-tiles {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 10px;
  margin-bottom: 12px;
}
.stat-tile {
  background: var(--surface);
  border: 1px solid var(--border);
  border-radius: var(--radius);
  padding: 14px;
  display: flex;
  flex-direction: column;
  gap: 3px;
}
.stat-tile-icon  { font-size: 20px; margin-bottom: 4px; }
.stat-tile-val   { font-family: var(--font-head); font-size: 26px; font-weight: 800; line-height: 1; }
.stat-tile-label { font-size: 12px; font-weight: 600; color: var(--text2); margin-top: 4px; }
.stat-tile-hint  { font-size: 11px; color: var(--muted); }

/* Formula Box */
.formula-box {
  background: var(--surface);
  border: 1px solid var(--border);
  border-radius: var(--radius);
  padding: 14px 16px;
  margin-bottom: 12px;
}
.formula-box-title {
  font-size: 11px;
  color: var(--muted);
  text-transform: uppercase;
  letter-spacing: .07em;
  margin-bottom: 12px;
}
.formula-row {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 4px;
}
.formula-item {
  text-align: center;
  flex: 1;
}
.formula-item.result {
  background: var(--surface2);
  border-radius: var(--radius-xs);
  padding: 6px 4px;
}
.formula-num { font-family: var(--font-head); font-size: 18px; font-weight: 800; line-height: 1; }
.formula-sub { font-size: 10px; color: var(--muted); margin-top: 3px; }
.formula-op  { font-size: 18px; font-weight: 700; color: var(--muted); flex: 0 0 auto; padding: 0 2px; }

/* Money Card */
.money-card {
  background: var(--surface);
  border: 1px solid var(--border);
  border-radius: var(--radius);
  padding: 16px;
  margin-bottom: 12px;
}
.money-title {
  font-family: var(--font-head);
  font-size: 13px;
  font-weight: 700;
  color: var(--text2);
  margin-bottom: 12px;
}
.money-row {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 10px 0;
  border-bottom: 1px solid var(--border);
}
.money-left {
  display: flex;
  align-items: center;
  gap: 10px;
}
.money-icon { font-size: 18px; }
.money-label { font-size: 14px; font-weight: 500; }
.money-hint  { font-size: 11px; color: var(--muted); margin-top: 2px; }
.money-val   { font-family: var(--font-head); font-weight: 700; font-size: 16px; }
.money-divider { height: 1px; background: var(--border); margin: 4px 0; }
.money-result {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 12px;
  border-radius: var(--radius-sm);
  margin-top: 8px;
}
.money-result.profit { background: var(--green-dim); border: 1px solid rgba(76,175,121,.25); }
.money-result.loss   { background: var(--red-dim);   border: 1px solid rgba(224,82,82,.25); }
.money-result-left { display: flex; align-items: center; gap: 10px; }
.money-result-icon  { font-size: 22px; }
.money-result-label { font-family: var(--font-head); font-size: 15px; font-weight: 700; }
.money-result-hint  { font-size: 11px; color: var(--muted); margin-top: 2px; }
.money-result-val   { font-family: var(--font-head); font-size: 22px; font-weight: 800; }
.money-result.profit .money-result-val { color: var(--green); }
.money-result.loss   .money-result-val { color: var(--red); }

.inv-thead {
  display: grid;
  grid-template-columns: 2fr 1fr 1fr 1fr 1fr;
  gap: 4px;
  padding: 9px 12px;
  background: var(--surface2);
  border-radius: var(--radius-sm) var(--radius-sm) 0 0;
  border: 1px solid var(--border);
  font-size: 10px;
  color: var(--muted);
  text-transform: uppercase;
  letter-spacing: .07em;
}
.col-c { text-align: center; }
.inv-row {
  display: grid;
  grid-template-columns: 2fr 1fr 1fr 1fr 1fr;
  gap: 4px;
  padding: 11px 12px;
  background: var(--surface);
  border: 1px solid var(--border);
  border-top: none;
  align-items: center;
  cursor: pointer;
  transition: background .15s;
}
.inv-row:last-child { border-radius: 0 0 var(--radius-sm) var(--radius-sm); }
.inv-row:hover { background: var(--surface2); }
.inv-row.expired { background: rgba(224,82,82,.04); }
.inv-cell { display: flex; align-items: center; }
.inv-cell.col-c { justify-content: center; font-size: 13px; font-weight: 500; }
.inv-name { font-weight: 500; font-size: 13px; }
.inv-num  { font-family: var(--font-head); font-weight: 700; font-size: 16px; }

.detail-section-title {
  font-size: 11px;
  font-weight: 600;
  color: var(--muted);
  text-transform: uppercase;
  letter-spacing: .07em;
  margin-bottom: 6px;
}
.detail-grid { display: flex; flex-direction: column; }
.detail-row {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 9px 0;
  border-bottom: 1px solid var(--border);
}
.detail-row:last-child { border-bottom: none; }
.detail-row.formula-row {
  background: var(--surface2);
  padding: 8px 12px;
  border-radius: var(--radius-xs);
  margin: 4px 0;
  border: none;
}
.detail-row.highlight-row { padding: 10px 0; }
.detail-label  { font-size: 13px; color: var(--text2); }
.detail-val    { font-family: var(--font-head); font-weight: 700; font-size: 15px; }
.formula-text  { font-family: var(--font-head); font-size: 12px; color: var(--accent); }

.batch-row {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 8px 0;
  border-bottom: 1px solid var(--border);
}
</style>
