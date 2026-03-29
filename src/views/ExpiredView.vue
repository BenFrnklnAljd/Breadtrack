<template>
  <div class="page">
    <div class="page-title">Expired Products</div>
    <div class="page-subtitle">Read-only pullout log · loss counted in reports</div>
    <div class="exp-summary">
      <div class="exp-sum-item"><div class="exp-sum-label">Expired Units</div><div class="exp-sum-val text-red">{{ inv.totalExpiredUnits }}</div></div>
      <div class="exp-sum-div"></div>
      <div class="exp-sum-item"><div class="exp-sum-label">Total Loss</div><div class="exp-sum-val text-red">₱{{ inv.totalLossValue.toFixed(2) }}</div></div>
      <div class="exp-sum-div"></div>
      <div class="exp-sum-item"><div class="exp-sum-label">Products</div><div class="exp-sum-val">{{ affectedProducts }}</div></div>
    </div>
    <div class="form-group"><label class="field-label">Filter by Month</label><select class="field-input" v-model="filterMonth"><option value="">All Time</option><option v-for="m in months" :key="m.value" :value="m.value">{{ m.label }}</option></select></div>
    <div v-if="filterMonth" class="month-loss-banner"><span>📅 Loss for {{ filterMonthLabel }}</span><span class="text-red" style="font-family:var(--font-head);font-weight:700;font-size:16px">₱{{ monthlyLoss.toFixed(2) }}</span></div>
    <div class="readonly-notice">🔒 These records are read-only. Original production entries remain intact.</div>
    <div v-if="filteredLog.length === 0" class="empty-state"><div class="empty-icon">✅</div><div class="empty-text">{{ filterMonth ? 'No expired products this month.' : 'No expired products yet.' }}</div></div>
    <div v-for="e in filteredLog" :key="e.batchId" class="exp-card">
      <div class="exp-card-top">
        <div class="exp-card-info"><div class="exp-name">{{ e.name }}</div><div class="date-meta">Produced: {{ e.date }} · {{ e.staffName }}</div></div>
        <div class="exp-tag">Expired</div>
      </div>
      <div class="exp-breakdown">
        <div class="exp-row"><span class="exp-row-label">Originally Produced</span><span class="exp-row-val">{{ e.produced }} units</span></div>
        <div class="exp-row"><span class="exp-row-label">Sold Before Expiry</span><span class="exp-row-val text-green">{{ e.sold }} units</span></div>
        <div class="exp-row"><span class="exp-row-label">Expired / Pullout</span><span class="exp-row-val text-red">{{ e.expiredQty }} units</span></div>
        <div class="exp-row exp-row-total"><span class="exp-row-label" style="font-weight:600;color:var(--text)">Loss Value</span><span class="exp-row-val text-red" style="font-size:15px">₱{{ e.lossValue.toFixed(2) }}</span></div>
      </div>
      <div class="exp-footer">{{ e.daysOld }} days since production</div>
    </div>
  </div>
</template>
<script setup>
import { ref, computed } from 'vue'
import { useInventoryStore } from '@/stores/inventory'
const inv = useInventoryStore()
const filterMonth = ref('')
const months = computed(() => { const seen = new Set(); inv.expiredBatchLog.forEach(e => seen.add(e.date.slice(0,7))); return [...seen].sort((a,b) => b.localeCompare(a)).map(m => { const [y,mo] = m.split('-'); const label = new Date(+y,+mo-1,1).toLocaleDateString('en-PH',{month:'long',year:'numeric'}); return { value: m, label } }) })
const filterMonthLabel = computed(() => months.value.find(m => m.value === filterMonth.value)?.label || '')
const filteredLog = computed(() => filterMonth.value ? inv.expiredBatchLog.filter(e => e.date.startsWith(filterMonth.value)) : inv.expiredBatchLog)
const monthlyLoss = computed(() => filteredLog.value.reduce((s,e) => s+e.lossValue,0))
const affectedProducts = computed(() => new Set(inv.expiredBatchLog.map(e => e.productId)).size)
</script>
<style scoped>
.exp-summary { display: flex; flex-direction: row; align-items: center; background: var(--surface); border: 1px solid var(--border); border-radius: var(--radius); padding: 16px 12px; margin-bottom: 16px; }
.exp-sum-item { flex: 1; display: flex; flex-direction: column; align-items: center; gap: 5px; }
.exp-sum-div { width: 1px; height: 36px; background: var(--border); flex-shrink: 0; margin: 0 6px; }
.exp-sum-label { font-size: 9px; color: var(--muted); text-transform: uppercase; letter-spacing: .07em; text-align: center; }
.exp-sum-val { font-family: var(--font-head); font-size: 20px; font-weight: 800; line-height: 1; }
.month-loss-banner { display: flex; justify-content: space-between; align-items: center; background: var(--red-dim); border: 1px solid rgba(224,82,82,.2); border-radius: var(--radius-sm); padding: 11px 14px; margin-bottom: 12px; font-size: 13px; color: var(--text2); }
.readonly-notice { background: var(--surface2); border: 1px solid var(--border); border-left: 3px solid var(--accent); border-radius: var(--radius-xs); padding: 10px 14px; font-size: 12px; color: var(--muted); margin-bottom: 16px; line-height: 1.6; }
.exp-card { background: var(--surface); border: 1px solid rgba(224,82,82,.2); border-radius: var(--radius); overflow: hidden; margin-bottom: 12px; }
.exp-card-top { display: flex; justify-content: space-between; align-items: flex-start; padding: 14px 14px 12px; border-bottom: 1px solid var(--border); }
.exp-card-info { flex: 1; padding-right: 10px; }
.exp-name { font-family: var(--font-head); font-size: 15px; font-weight: 700; margin-bottom: 3px; }
.exp-tag { background: var(--red-dim); color: var(--red); border: 1px solid rgba(224,82,82,.25); border-radius: 20px; font-size: 11px; font-weight: 600; padding: 3px 10px; flex-shrink: 0; }
.exp-breakdown { background: var(--surface2); }
.exp-row { display: flex; justify-content: space-between; align-items: center; padding: 9px 14px; border-bottom: 1px solid var(--border); font-size: 13px; }
.exp-row:last-child { border-bottom: none; }
.exp-row.exp-row-total { background: var(--surface); padding: 11px 14px; }
.exp-row-label { color: var(--text2); }
.exp-row-val { font-family: var(--font-head); font-weight: 700; font-size: 13px; }
.exp-footer { padding: 7px 14px; font-size: 11px; color: var(--muted); border-top: 1px solid var(--border); text-align: right; background: var(--surface); }
</style>