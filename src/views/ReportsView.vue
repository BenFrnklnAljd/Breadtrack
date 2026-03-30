<template>
  <div class="page">
    <div class="page-title">Reports</div>
    <div class="page-subtitle">Analytics & exports</div>
    <div class="period-row">
      <button v-for="p in periods" :key="p.v" :class="['period-btn', period === p.v ? 'active' : '']" @click="period = p.v">{{ p.label }}</button>
    </div>
    <Transition name="fade">
      <div v-if="period === 'monthly'" class="month-picker">
        <div class="month-picker-top"><span style="font-size:13px;color:var(--text2)"> Select Month</span><span style="font-family:var(--font-head);font-size:13px;font-weight:700;color:var(--accent)">{{ selectedMonthLabel }}</span></div>
        <input class="field-input" type="month" v-model="selectedMonth" :max="currentYM" />
      </div>
    </Transition>
    <div class="form-group"><label class="field-label">Filter by Product</label><select class="field-input" v-model="productFilter"><option value="">All Products</option><option v-for="p in inv.products" :key="p.id" :value="p.id">{{ p.name }}</option></select></div>
    <div class="period-badge"><span>Showing: <strong>{{ activePeriodLabel }}</strong></span><span style="color:var(--muted);font-size:12px">{{ filtered.length }} sales</span></div>
    <div class="stats-grid">
      <div class="stat-card"><div class="stat-label">Revenue</div><div class="stat-val val-accent">₱{{ stats.revenue.toFixed(2) }}</div></div>
      <div class="stat-card"><div class="stat-label">Units Sold</div><div class="stat-val">{{ stats.units }}</div></div>
      <div class="stat-card"><div class="stat-label">Loss</div><div class="stat-val val-red">₱{{ stats.loss.toFixed(2) }}</div></div>
      <div class="stat-card"><div class="stat-label">Net Profit</div><div class="stat-val" :class="stats.net >= 0 ? 'val-green' : 'val-red'">₱{{ stats.net.toFixed(2) }}</div></div>
    </div>
    <div class="card" style="margin-bottom:12px">
      <div style="font-family:var(--font-head);font-size:13px;font-weight:700;margin-bottom:14px">Sales by Product</div>
      <div v-if="byProduct.length === 0" style="font-size:13px;color:var(--muted)">No data.</div>
      <div v-else class="bar-chart">
        <div v-for="item in byProduct" :key="item.id" class="bar-row"><div class="bar-label">{{ item.name }}</div><div class="bar-track"><div class="bar-fill sales" :style="{ width: item.pct + '%' }">{{ item.revenue > 0 ? '₱'+item.revenue.toFixed(0) : '' }}</div></div></div>
      </div>
    </div>
    <div class="card" style="margin-bottom:16px">
      <div style="font-family:var(--font-head);font-size:13px;font-weight:700;margin-bottom:12px">Sales Log</div>
      <div v-if="filtered.length === 0" style="font-size:13px;color:var(--muted)">No sales in this period.</div>
      <div v-else>
        <div class="log-head"><span>Date</span><span>Product</span><span style="text-align:right">Amount</span></div>
        <div v-for="s in filtered.slice().reverse()" :key="s.id" class="log-row"><span style="font-size:11px;color:var(--muted)">{{ s.date }}</span><span style="font-weight:500;overflow:hidden;text-overflow:ellipsis;white-space:nowrap">{{ inv.productName(s.productId) }}</span><span style="font-family:var(--font-head);font-weight:700;color:var(--accent);text-align:right">₱{{ (s.qty*s.price).toFixed(2) }}</span></div>
      </div>
    </div>
    <div class="section-label">Export</div>
    <div style="display:flex;flex-direction:column;gap:8px;margin-bottom:32px">
      <button class="btn btn-primary btn-block" :disabled="exporting" @click="doExport">{{ exporting ? '⏳ Generating…' : '📄 Export PDF — ' + activePeriodLabel }}</button>
      <button class="btn btn-ghost btn-block" @click="showReceipt = true">🧾 Today's Receipt</button>
    </div>
    <BaseModal v-model="showReceipt" title="Today's Receipt">
      <div class="receipt">
        <div class="receipt-header"><div class="receipt-logo">🥐 PerishTrack</div><div class="receipt-meta">{{ todayLabel() }}</div><div class="receipt-meta">{{ auth.currentUser?.name }}</div></div>
        <div v-if="inv.todaySales.length === 0" style="text-align:center;padding:20px;color:var(--muted);font-size:13px">No sales today.</div>
        <div v-for="s in inv.todaySales" :key="s.id" class="receipt-line"><span>{{ inv.productName(s.productId) }} × {{ s.qty }}</span><span>₱{{ (s.qty*s.price).toFixed(2) }}</span></div>
        <div v-if="inv.todaySales.length > 0" class="receipt-total"><span>Total</span><span>₱{{ inv.todayRevenue.toFixed(2) }}</span></div>
      </div>
      <button class="btn btn-ghost btn-block" style="margin-top:14px" @click="showReceipt = false">Close</button>
    </BaseModal>
  </div>
</template>
<script setup>
import { ref, computed } from 'vue'
import { useInventoryStore } from '@/stores/inventory'
import { useAuthStore } from '@/stores/auth'
import { useToastStore } from '@/stores/toast'
import { todayLabel, today } from '@/utils/date'
import { usePdf } from '@/composables/usePdf'
import BaseModal from '@/components/ui/BaseModal.vue'
const inv = useInventoryStore(); const auth = useAuthStore(); const toast = useToastStore()
const { exportReport } = usePdf()
const period = ref('daily'); const productFilter = ref(''); const showReceipt = ref(false); const exporting = ref(false)
const currentYM = today().slice(0,7); const selectedMonth = ref(currentYM)
const periods = [{ v:'daily', label:'Today' }, { v:'weekly', label:'This Week' }, { v:'monthly', label:'Monthly' }]
const selectedMonthLabel = computed(() => { if (!selectedMonth.value) return ''; const [y,m] = selectedMonth.value.split('-'); return new Date(+y,+m-1,1).toLocaleDateString('en-PH',{month:'long',year:'numeric'}) })
const activeTargetMonth = computed(() => period.value === 'monthly' ? selectedMonth.value : null)
const activePeriodLabel = computed(() => { if (period.value==='daily') return 'Today'; if (period.value==='weekly') return 'This Week'; return selectedMonthLabel.value })
const filtered = computed(() => inv.getFilteredSales({ period: period.value, productId: productFilter.value, targetMonth: activeTargetMonth.value }))
const stats = computed(() => { const revenue = filtered.value.reduce((s,x) => s+x.qty*x.price,0); const units = filtered.value.reduce((s,x) => s+x.qty,0); const loss = inv.totalLossValue; return { revenue, units, loss, net: revenue-loss } })
const byProduct = computed(() => inv.getSalesByProduct(filtered.value))
async function doExport() { exporting.value=true; try { await exportReport(period.value, productFilter.value, activeTargetMonth.value); toast.success('PDF downloaded!') } catch { toast.error('Export failed.') } finally { exporting.value=false } }
</script>
<style scoped>
.period-row { display: flex; gap: 6px; margin-bottom: 14px; }
.period-btn { flex: 1; padding: 10px 6px; border-radius: var(--radius-sm); border: 1px solid var(--border); background: var(--surface2); color: var(--muted); font-size: 13px; font-weight: 500; cursor: pointer; font-family: var(--font-body); transition: all .15s; }
.period-btn.active { background: var(--accent); border-color: var(--accent); color: #0a0a09; font-weight: 700; }
.month-picker { background: var(--surface); border: 1px solid var(--border); border-radius: var(--radius); padding: 14px; margin-bottom: 14px; }
.month-picker-top { display: flex; justify-content: space-between; align-items: center; margin-bottom: 10px; }
.period-badge { display: flex; justify-content: space-between; align-items: center; background: var(--surface2); border: 1px solid var(--border); border-radius: var(--radius-xs); padding: 9px 14px; margin-bottom: 14px; font-size: 13px; color: var(--text2); }
.log-head { display: grid; grid-template-columns: 80px 1fr 80px; font-size: 10px; color: var(--muted); text-transform: uppercase; letter-spacing: .07em; padding: 6px 0; border-bottom: 1px solid var(--border); margin-bottom: 4px; }
.log-row { display: grid; grid-template-columns: 80px 1fr 80px; padding: 8px 0; border-bottom: 1px solid var(--border); font-size: 13px; align-items: center; gap: 8px; }
</style>
