<template>
  <div class="page">
    <div class="page-title">Expired Products</div>
    <div class="page-subtitle">Read-only pullout log · loss counted in reports</div>

<<<<<<< HEAD
    <!-- Summary Bar -->
=======
    <!-- Summaary -->
>>>>>>> 9f32d946c7d9c2427cef210d59a016cb80a1671a
    <div class="exp-summary">
      <div class="exp-sum-item">
        <div class="exp-sum-label">Expired Units</div>
        <div class="exp-sum-val red">{{ inv.totalExpiredUnits }}</div>
      </div>
      <div class="exp-sum-divider"></div>
      <div class="exp-sum-item">
        <div class="exp-sum-label">Total Loss</div>
        <div class="exp-sum-val red">&#8369;{{ inv.totalLossValue.toFixed(2) }}</div>
      </div>
      <div class="exp-sum-divider"></div>
      <div class="exp-sum-item">
        <div class="exp-sum-label">Products</div>
        <div class="exp-sum-val">{{ affectedProducts }}</div>
      </div>
    </div>

    <!-- Month filter -->
    <div class="form-group">
      <label class="field-label">Filter by Month</label>
      <select class="field-input" v-model="filterMonth">
        <option value="">All Time</option>
        <option v-for="m in availableMonths" :key="m.value" :value="m.value">
          {{ m.label }}
        </option>
      </select>
    </div>

    <!-- Monthly loss banner -->
    <div v-if="filterMonth" class="monthly-banner">
      <span>&#128197; Loss for {{ filterMonthLabel }}</span>
      <span class="monthly-banner-val">&#8369;{{ monthlyLoss.toFixed(2) }}</span>
    </div>

    <!-- Read-only notice -->
    <div class="readonly-notice">
       Expired records are read-only. Original production records remain intact.
    </div>

    <!-- Empty state -->
    <div v-if="filteredLog.length === 0" class="empty-state">
      <div class="empty-icon">&#9989;</div>
      <div class="empty-text">
        {{ filterMonth ? 'No expired products for this month.' : 'No expired products yet.' }}
      </div>
    </div>

    <!-- Expired Batch Cards -->
    <div v-for="entry in filteredLog" :key="entry.batchId" class="exp-card">

      <div class="exp-card-header">
        <div class="exp-card-header-left">
          <div class="exp-name">{{ entry.name }}</div>
          <div class="exp-meta">Produced: {{ entry.date }} &middot; {{ entry.staffName }}</div>
        </div>
        <div class="exp-tag">Expired</div>
      </div>

      <div class="exp-table">
        <div class="exp-table-row">
          <span class="exp-table-label">Originally Produced</span>
          <span class="exp-table-val">{{ entry.produced }} units</span>
        </div>
        <div class="exp-table-row">
          <span class="exp-table-label">Sold Before Expiry</span>
          <span class="exp-table-val green">{{ entry.sold }} units</span>
        </div>
        <div class="exp-table-row">
          <span class="exp-table-label">Expired / Pullout</span>
          <span class="exp-table-val red">{{ entry.expiredQty }} units</span>
        </div>
        <div class="exp-table-row total-row">
          <span class="exp-table-label bold">Loss Value</span>
          <span class="exp-table-val red bold">&#8369;{{ entry.lossValue.toFixed(2) }}</span>
        </div>
      </div>

      <div class="exp-card-footer">
        {{ entry.daysOld }} days since production
      </div>

    </div>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'
import { useInventoryStore } from '@/stores/inventory'

const inv = useInventoryStore()
const filterMonth = ref('')

const availableMonths = computed(() => {
  const seen = new Set()
  inv.expiredBatchLog.forEach(e => seen.add(e.date.slice(0, 7)))
  return [...seen].sort((a, b) => b.localeCompare(a)).map(m => {
    const [yr, mo] = m.split('-')
    const label = new Date(parseInt(yr), parseInt(mo) - 1, 1)
      .toLocaleDateString('en-PH', { month: 'long', year: 'numeric' })
    return { value: m, label }
  })
})

const filterMonthLabel = computed(() =>
  availableMonths.value.find(m => m.value === filterMonth.value)?.label || ''
)

const filteredLog = computed(() =>
  filterMonth.value
    ? inv.expiredBatchLog.filter(e => e.date.startsWith(filterMonth.value))
    : inv.expiredBatchLog
)

const monthlyLoss = computed(() =>
  filteredLog.value.reduce((s, e) => s + e.lossValue, 0)
)

const affectedProducts = computed(() =>
  new Set(inv.expiredBatchLog.map(e => e.productId)).size
)
</script>

<style scoped>
/* ── Summary Bar ── */
.exp-summary {
  display: flex;
  flex-direction: row;
  align-items: center;
  background: var(--surface);
  border: 1px solid var(--border);
  border-radius: var(--radius);
  padding: 16px 12px;
  margin-bottom: 16px;
}
.exp-sum-item {
  flex: 1;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 5px;
}
.exp-sum-divider {
  width: 1px;
  height: 36px;
  background: var(--border);
  flex-shrink: 0;
  margin: 0 6px;
}
.exp-sum-label {
  font-size: 9px;
  color: var(--muted);
  text-transform: uppercase;
  letter-spacing: .07em;
  text-align: center;
}
.exp-sum-val {
  font-family: var(--font-head);
  font-size: 20px;
  font-weight: 800;
  line-height: 1;
  color: var(--text);
}
.exp-sum-val.red { color: var(--red); }

/* ── Monthly Banner ── */
.monthly-banner {
  display: flex;
  justify-content: space-between;
  align-items: center;
  background: var(--red-dim);
  border: 1px solid rgba(224,82,82,.25);
  border-radius: var(--radius-sm);
  padding: 11px 14px;
  margin-bottom: 12px;
  font-size: 13px;
  color: var(--text2);
}
.monthly-banner-val {
  font-family: var(--font-head);
  font-size: 16px;
  font-weight: 700;
  color: var(--red);
}

/* ── Read-only Notice ── */
.readonly-notice {
  background: var(--surface2);
  border: 1px solid var(--border);
  border-left: 3px solid var(--accent);
  border-radius: var(--radius-xs);
  padding: 10px 14px;
  font-size: 12px;
  color: var(--muted);
  margin-bottom: 16px;
  line-height: 1.6;
}

/* ── Expired Card ── */
.exp-card {
  background: var(--surface);
  border: 1px solid rgba(224,82,82,.22);
  border-radius: var(--radius);
  overflow: hidden;
  margin-bottom: 12px;
}
.exp-card-header {
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
  padding: 14px 16px 12px;
  border-bottom: 1px solid var(--border);
}
.exp-card-header-left {
  flex: 1;
  padding-right: 10px;
}
.exp-name {
  font-family: var(--font-head);
  font-size: 15px;
  font-weight: 700;
  color: var(--text);
  margin-bottom: 3px;
}
.exp-meta {
  font-size: 11px;
  color: var(--muted);
}
.exp-tag {
  background: var(--red-dim);
  color: var(--red);
  border: 1px solid rgba(224,82,82,.3);
  border-radius: 20px;
  font-size: 11px;
  font-weight: 600;
  padding: 3px 10px;
  white-space: nowrap;
  flex-shrink: 0;
}

/* ── Breakdown Table ── */
.exp-table {
  background: var(--surface2);
}
.exp-table-row {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 10px 16px;
  border-bottom: 1px solid var(--border);
  font-size: 13px;
}
.exp-table-row:last-child {
  border-bottom: none;
}
.exp-table-row.total-row {
  background: var(--surface);
  padding: 12px 16px;
}
.exp-table-label {
  color: var(--text2);
}
.exp-table-label.bold {
  font-weight: 600;
  color: var(--text);
}
.exp-table-val {
  font-family: var(--font-head);
  font-weight: 700;
  font-size: 13px;
  color: var(--text);
}
.exp-table-val.green { color: var(--green); }
.exp-table-val.red   { color: var(--red); }
.exp-table-val.bold  { font-size: 15px; }

<<<<<<< HEAD
/* ── Footer ── */
.exp-card-footer {
  padding: 8px 16px;
  font-size: 11px;
  color: var(--muted);
  border-top: 1px solid var(--border);
  text-align: right;
  background: var(--surface);
}
</style>
=======
.exp-footer { display: flex; justify-content: flex-end; }
</style>
>>>>>>> 9f32d946c7d9c2427cef210d59a016cb80a1671a
