<template>
  <div class="page">
    <div class="page-title">Expired Products</div>
    <div class="page-subtitle">Read-only pullout log · loss counted in reports</div>

    <!-- Summary -->
    <div class="exp-summary">
      <div class="exp-sum-item">
        <div class="exp-sum-label">Total Expired Items</div>
        <div class="exp-sum-val text-red">{{ inv.totalExpiredUnits }}</div>
      </div>
      <div class="exp-sum-divider"></div>
      <div class="exp-sum-item">
        <div class="exp-sum-label">Total Loss Value</div>
        <div class="exp-sum-val text-red">₱{{ inv.totalLossValue.toFixed(2) }}</div>
      </div>
      <div class="exp-sum-divider"></div>
      <div class="exp-sum-item">
        <div class="exp-sum-label">Affected Products</div>
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

    <!-- Monthly loss banner when filtered -->
    <div v-if="filterMonth" class="monthly-loss-banner">
      <span>📅 Loss for {{ filterMonthLabel }}</span>
      <span class="monthly-loss-val text-red">₱{{ monthlyLoss.toFixed(2) }}</span>
    </div>

    <!-- Notice -->
    <div class="readonly-notice">
      🔒 Expired records are read-only and cannot be edited or deleted. Original production records remain intact.
    </div>

    <!-- Empty state -->
    <div v-if="filteredLog.length === 0" class="empty-state">
      <div class="empty-icon">✅</div>
      <div class="empty-text">
        {{ filterMonth ? 'No expired products for this month.' : 'No expired products yet.' }}
      </div>
    </div>

    <!-- Expired Batch Cards -->
    <TransitionGroup name="fade">
      <div v-for="entry in filteredLog" :key="entry.batchId" class="exp-card">
        <div class="exp-card-top">
          <div>
            <div class="exp-product-name">{{ entry.name }}</div>
            <div class="date-meta">Produced: {{ entry.date }} · by {{ entry.staffName }}</div>
          </div>
          <div class="exp-badge">
            <span class="tag tag-expired">Expired</span>
          </div>
        </div>

        <!-- Computation breakdown -->
        <div class="exp-breakdown">
          <div class="exp-breakdown-row">
            <span class="exp-breakdown-label">Originally Produced</span>
            <span class="exp-breakdown-val">{{ entry.produced }} units</span>
          </div>
          <div class="exp-breakdown-row">
            <span class="exp-breakdown-label">Sold Before Expiry</span>
            <span class="exp-breakdown-val text-green">{{ entry.sold }} units</span>
          </div>
          <div class="exp-breakdown-row">
            <span class="exp-breakdown-label">Expired / Pullout</span>
            <span class="exp-breakdown-val text-red">{{ entry.expiredQty }} units</span>
          </div>
          <div class="exp-breakdown-row exp-breakdown-total">
            <span class="exp-breakdown-label" style="font-weight:600">Loss Value</span>
            <span class="exp-breakdown-val text-red" style="font-size:15px">₱{{ entry.lossValue.toFixed(2) }}</span>
          </div>
        </div>

        <div class="exp-footer">
          <span class="date-meta">{{ entry.daysOld }} days since production</span>
        </div>
      </div>
    </TransitionGroup>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'
import { useInventoryStore } from '@/stores/inventory'

const inv = useInventoryStore()

const filterMonth = ref('')

// All unique months from the expired log
const availableMonths = computed(() => {
  const seen = new Set()
  inv.expiredBatchLog.forEach(e => {
    const m = e.date.slice(0, 7)
    seen.add(m)
  })
  return [...seen].sort((a, b) => b.localeCompare(a)).map(m => {
    const [yr, mo] = m.split('-')
    const label = new Date(parseInt(yr), parseInt(mo) - 1, 1)
      .toLocaleDateString('en-PH', { month: 'long', year: 'numeric' })
    return { value: m, label }
  })
})

const filterMonthLabel = computed(() => {
  const found = availableMonths.value.find(m => m.value === filterMonth.value)
  return found?.label || ''
})

const filteredLog = computed(() => {
  if (!filterMonth.value) return inv.expiredBatchLog
  return inv.expiredBatchLog.filter(e => e.date.startsWith(filterMonth.value))
})

const monthlyLoss = computed(() =>
  filteredLog.value.reduce((s, e) => s + e.lossValue, 0)
)

const affectedProducts = computed(() =>
  new Set(inv.expiredBatchLog.map(e => e.productId)).size
)
</script>

<style scoped>
/* Summary Bar */
.exp-summary {
  display: flex;
  background: var(--surface);
  border: 1px solid var(--border);
  border-radius: var(--radius);
  padding: 14px 16px;
  margin-bottom: 16px;
  gap: 0;
}
.exp-sum-item { flex: 1; text-align: center; }
.exp-sum-divider { width: 1px; background: var(--border); margin: 0 8px; }
.exp-sum-label { font-size: 10px; color: var(--muted); text-transform: uppercase; letter-spacing: .06em; margin-bottom: 4px; }
.exp-sum-val   { font-family: var(--font-head); font-size: 20px; font-weight: 800; }

/* Monthly Banner */
.monthly-loss-banner {
  display: flex;
  justify-content: space-between;
  align-items: center;
  background: var(--red-dim);
  border: 1px solid rgba(224,82,82,.25);
  border-radius: var(--radius-sm);
  padding: 10px 14px;
  margin-bottom: 12px;
  font-size: 13px;
  color: var(--text2);
}
.monthly-loss-val {
  font-family: var(--font-head);
  font-weight: 700;
  font-size: 16px;
}

/* Read-only Notice */
.readonly-notice {
  background: var(--surface2);
  border: 1px solid var(--border);
  border-radius: var(--radius-xs);
  padding: 10px 14px;
  font-size: 12px;
  color: var(--muted);
  margin-bottom: 14px;
  line-height: 1.5;
}

/* Expired Card */
.exp-card {
  background: var(--surface);
  border: 1px solid rgba(224,82,82,.25);
  border-radius: var(--radius);
  padding: 14px 16px;
  margin-bottom: 10px;
}
.exp-card-top {
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
  margin-bottom: 12px;
}
.exp-product-name {
  font-weight: 600;
  font-size: 15px;
  margin-bottom: 3px;
  color: var(--text);
}

/* Breakdown */
.exp-breakdown {
  background: var(--surface2);
  border-radius: var(--radius-xs);
  padding: 10px 12px;
  display: flex;
  flex-direction: column;
  gap: 0;
  margin-bottom: 10px;
}
.exp-breakdown-row {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 6px 0;
  border-bottom: 1px solid var(--border);
  font-size: 13px;
}
.exp-breakdown-row:last-child { border-bottom: none; }
.exp-breakdown-row.exp-breakdown-total {
  padding-top: 8px;
  margin-top: 2px;
}
.exp-breakdown-label { color: var(--text2); }
.exp-breakdown-val   { font-family: var(--font-head); font-weight: 700; font-size: 13px; }

.exp-footer { display: flex; justify-content: flex-end; }
</style>