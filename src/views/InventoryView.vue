<template>
  <div class="page">
    <div class="page-title">Stock Overview</div>
    <div class="page-subtitle">Full inventory computation</div>
    <div class="summary-block">
      <div class="summary-block-title">📦 All-Time Totals</div>
      <div class="list-row"><span class="list-label">Total Produced</span><span class="list-val">{{ grand.produced }} units</span></div>
      <div class="list-row"><span class="list-label">Total Sold</span><span class="list-val text-green">{{ grand.sold }} units</span></div>
      <div class="list-row"><span class="list-label">Expired / Pullout</span><span class="list-val text-red">{{ grand.expired }} units</span></div>
      <div class="list-row" style="padding-top:14px"><span class="list-label" style="font-weight:600;color:var(--text)">Remaining Stock</span><span class="list-val text-accent" style="font-size:20px">{{ grand.remaining }} units</span></div>
      <div class="divider"></div>
      <div class="list-row"><span class="list-label">Revenue</span><span class="list-val text-green">₱{{ grand.revenue.toFixed(2) }}</span></div>
      <div class="list-row"><span class="list-label">Loss (Expired)</span><span class="list-val text-red">₱{{ grand.loss.toFixed(2) }}</span></div>
      <div class="list-row" style="padding-top:14px"><span class="list-label" style="font-weight:600;color:var(--text)">Net Profit</span><span class="list-val" style="font-size:18px" :class="grand.net >= 0 ? 'text-green' : 'text-red'">₱{{ grand.net.toFixed(2) }}</span></div>
    </div>
    <div class="form-group"><label class="field-label">Category</label><select class="field-input" v-model="filterCat"><option value="">All</option><option value="food">🍞 Food</option><option value="beverage">☕ Beverage</option></select></div>
    <div class="section-label">Per Product</div>
    <div v-if="filtered.length === 0" class="empty-state"><div class="empty-icon">📋</div><div class="empty-text">No products found.</div></div>
    <div v-else class="inv-table">
      <div class="inv-head"><span>Product</span><span class="c">Prod</span><span class="c">Sold</span><span class="c">Exp</span><span class="c">Bal</span></div>
      <div v-for="p in filtered" :key="p.id" class="inv-row" :class="{ 'inv-row-expired': inv.isExpired(p.id) }" @click="open(p)">
        <div class="inv-name-cell"><div class="inv-name">{{ p.name }}</div><span :class="p.category === 'food' ? 'tag tag-food' : 'tag tag-bev'" style="font-size:10px;margin-top:2px">{{ p.category }}</span></div>
        <span class="c">{{ inv.totalProduced(p.id) }}</span>
        <span class="c text-green">{{ inv.totalSold(p.id) }}</span>
        <span class="c text-red">{{ inv.expiredUnits(p.id) }}</span>
        <span class="c"><strong :class="inv.isExpired(p.id) ? 'text-red' : inv.currentBalance(p.id) > 0 ? 'text-green' : 'text-muted'">{{ inv.currentBalance(p.id) }}</strong></span>
      </div>
    </div>
    <BaseModal v-model="showModal" :title="selected?.name">
      <div v-if="selected">
        <div style="display:flex;gap:8px;flex-wrap:wrap;margin-bottom:16px">
          <span :class="selected.category === 'food' ? 'tag tag-food' : 'tag tag-bev'">{{ selected.category }}</span>
          <span class="price-tag">₱{{ selected.price }} / unit</span>
          <span v-if="inv.isExpired(selected.id)" class="tag tag-expired">Has Expired</span>
          <span v-else class="tag tag-ok">OK</span>
        </div>
        <div class="detail-group">
          <div class="detail-group-title">Inventory</div>
          <div class="list-row"><span class="list-label">Total Produced</span><span class="list-val">{{ inv.totalProduced(selected.id) }}</span></div>
          <div class="list-row"><span class="list-label">Total Sold</span><span class="list-val text-green">{{ inv.totalSold(selected.id) }}</span></div>
          <div class="list-row"><span class="list-label">Expired</span><span class="list-val text-red">{{ inv.expiredUnits(selected.id) }}</span></div>
          <div class="list-row"><span class="list-label formula">{{ inv.totalProduced(selected.id) }} − {{ inv.totalSold(selected.id) }} − {{ inv.expiredUnits(selected.id) }}</span><span class="list-val text-accent" style="font-size:20px">= {{ inv.currentBalance(selected.id) }}</span></div>
        </div>
        <div class="detail-group" style="margin-top:12px">
          <div class="detail-group-title">Financial</div>
          <div class="list-row"><span class="list-label">Revenue</span><span class="list-val text-green">₱{{ productRevenue(selected.id).toFixed(2) }}</span></div>
          <div class="list-row"><span class="list-label">Loss</span><span class="list-val text-red">₱{{ (inv.expiredUnits(selected.id) * selected.price).toFixed(2) }}</span></div>
          <div class="list-row"><span class="list-label" style="font-weight:600">Net</span><span class="list-val" :class="netProfit(selected) >= 0 ? 'text-green' : 'text-red'" style="font-size:18px">₱{{ netProfit(selected).toFixed(2) }}</span></div>
        </div>
        <button class="btn btn-ghost btn-block" style="margin-top:16px" @click="showModal = false">Close</button>
      </div>
    </BaseModal>
  </div>
</template>
<script setup>
import { ref, computed } from 'vue'
import { useInventoryStore } from '@/stores/inventory'
import BaseModal from '@/components/ui/BaseModal.vue'
const inv = useInventoryStore()
const filterCat = ref(''); const showModal = ref(false); const selected = ref(null)
const filtered = computed(() => inv.activeProducts.filter(p => !filterCat.value || p.category === filterCat.value))
const grand = computed(() => { const produced = inv.activeProducts.reduce((s,p) => s+inv.totalProduced(p.id),0); const sold = inv.activeProducts.reduce((s,p) => s+inv.totalSold(p.id),0); const expired = inv.activeProducts.reduce((s,p) => s+inv.expiredUnits(p.id),0); const remaining = inv.activeProducts.reduce((s,p) => s+inv.currentBalance(p.id),0); const revenue = inv.sales.reduce((s,x) => s+x.qty*x.price,0); const loss = inv.totalLossValue; return { produced, sold, expired, remaining, revenue, loss, net: revenue-loss } })
function open(p) { selected.value = p; showModal.value = true }
function productRevenue(pid) { return inv.sales.filter(s => s.productId === pid).reduce((s,x) => s+x.qty*x.price,0) }
function netProfit(p) { return productRevenue(p.id) - inv.expiredUnits(p.id) * p.price }
</script>
<style scoped>
.summary-block { background: var(--surface); border: 1px solid var(--border); border-radius: var(--radius); padding: 16px; margin-bottom: 16px; }
.summary-block-title { font-family: var(--font-head); font-size: 13px; font-weight: 700; color: var(--text2); margin-bottom: 8px; }
.inv-table { border-radius: var(--radius); overflow: hidden; border: 1px solid var(--border); }
.inv-head { display: grid; grid-template-columns: 2fr 1fr 1fr 1fr 1fr; padding: 8px 12px; background: var(--surface2); font-size: 10px; color: var(--muted); text-transform: uppercase; letter-spacing: .07em; }
.inv-row { display: grid; grid-template-columns: 2fr 1fr 1fr 1fr 1fr; padding: 11px 12px; background: var(--surface); border-top: 1px solid var(--border); align-items: center; cursor: pointer; transition: background .15s; font-size: 13px; }
.inv-row:hover { background: var(--surface2); }
.inv-row.inv-row-expired { background: rgba(224,82,82,.04); }
.c { text-align: center; font-weight: 500; }
.inv-name-cell { display: flex; flex-direction: column; }
.inv-name { font-weight: 600; font-size: 13px; }
.detail-group { background: var(--surface2); border-radius: var(--radius-sm); padding: 12px; }
.detail-group-title { font-size: 10px; font-weight: 600; color: var(--muted); text-transform: uppercase; letter-spacing: .08em; margin-bottom: 4px; }
.formula { font-family: var(--font-head); font-size: 12px; color: var(--accent); }
</style>