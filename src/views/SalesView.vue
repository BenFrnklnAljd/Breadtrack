<template>
  <div class="page">
    <div class="page-title">Sales</div>
    <div class="page-subtitle">Record transactions</div>
    <div class="panel">
      <div class="panel-title">+ New Sale</div>
      <div class="form-group"><label class="field-label">Product</label><select class="field-input" v-model="form.productId" @change="onProduct"><option value="">Select product…</option><option v-for="p in sellable" :key="p.id" :value="p.id">{{ p.name }} ({{ inv.sellableBalance(p.id) }} available)</option></select></div>
      <div class="form-row">
        <div class="form-group"><label class="field-label">Qty Sold</label><input class="field-input" type="number" v-model="form.qty" placeholder="0" min="1" /></div>
        <div class="form-group"><label class="field-label">Unit Price (₱)</label><input class="field-input" type="number" v-model="form.price" placeholder="auto" /></div>
      </div>
      <div class="form-group"><label class="field-label">Sale Date</label><input class="field-input" type="date" v-model="form.date" /></div>
      <div v-if="form.productId && form.qty && form.price" class="total-preview"><span>Total</span><span class="total-val">₱{{ (form.qty * form.price).toFixed(2) }}</span></div>
      <p v-if="err" class="form-error">{{ err }}</p>
      <button class="btn btn-primary btn-block" @click="record">Record Sale</button>
    </div>
    
    <div class="section-label">History ({{ inv.sales.length }})</div>
    <div v-if="inv.sales.length === 0" class="empty-state"><div class="empty-icon">🛒</div><div class="empty-text">No sales yet.</div></div>
    <div v-for="s in inv.sortedSales" :key="s.id" class="sale-card">
      <div class="sale-left"><div class="sale-name">{{ inv.productName(s.productId) }}</div><div class="date-meta">{{ s.date }} · {{ s.staffName }}</div></div>
      <div class="sale-right"><div class="sale-amount">₱{{ (s.qty * s.price).toFixed(2) }}</div><div style="font-size:11px;color:var(--muted)">{{ s.qty }} × ₱{{ s.price }}</div><button class="btn btn-xs" style="background:var(--red-dim);color:var(--red);border:none;margin-top:4px" @click="confirmDel(s)">🗑</button></div>
    </div>
    <BaseModal v-model="showDel" title="Delete Sale?">
      <p style="font-size:14px;color:var(--text2);margin-bottom:20px">Delete <strong>{{ delTarget?.qty }} × {{ inv.productName(delTarget?.productId) }}</strong>? Balance will be restored.</p>
      <div class="panel-actions"><button class="btn btn-danger btn-sm" @click="doDel">Delete</button><button class="btn btn-ghost btn-sm" @click="showDel = false">Cancel</button></div>
    </BaseModal>
  </div>
</template>
<script setup>
import { ref, reactive, computed } from 'vue'
import { useInventoryStore } from '@/stores/inventory'
import { useAuthStore } from '@/stores/auth'
import { useToastStore } from '@/stores/toast'
import { today, todayLabel } from '@/utils/date'
import { usePdf } from '@/composables/usePdf'
import BaseModal from '@/components/ui/BaseModal.vue'
const inv = useInventoryStore(); const auth = useAuthStore(); const toast = useToastStore()
const { exportReceipt } = usePdf()
const err = ref(''); const showDel = ref(false); const delTarget = ref(null)
const form = reactive({ productId: '', qty: '', price: '', date: today() })
const sellable = computed(() => inv.activeProducts.filter(p => inv.sellableBalance(p.id) > 0))
function onProduct() { const p = inv.getProduct(form.productId); if (p) form.price = p.price }
function record() { err.value = ''; if (!form.productId) { err.value = 'Select a product.'; return }; if (!form.qty || parseInt(form.qty) < 1) { err.value = 'Enter quantity.'; return }; if (!form.price || parseFloat(form.price) <= 0) { err.value = 'Enter price.'; return }; const bal = inv.sellableBalance(form.productId); if (parseInt(form.qty) > bal) { err.value = `Only ${bal} units available.`; return }; inv.addSale({ ...form, staffName: auth.currentUser.name }); toast.success('Sale recorded!'); Object.assign(form, { productId: '', qty: '', price: '', date: today() }) }
function downloadReceipt() { exportReceipt(inv.todaySales, auth.currentUser.name); toast.success('Receipt downloaded.') }
function confirmDel(s) { delTarget.value = s; showDel.value = true }
function doDel() { inv.deleteSale(delTarget.value.id); toast.info('Deleted.'); showDel.value = false }
</script>
<style scoped>
.total-preview { display: flex; justify-content: space-between; align-items: center; background: var(--accent-dim); border: 1px solid rgba(245,166,35,.2); border-radius: var(--radius-xs); padding: 10px 14px; margin-bottom: 12px; font-size: 14px; color: var(--text2); }
.total-val { font-family: var(--font-head); font-size: 18px; font-weight: 800; color: var(--accent); }
.sale-card { background: var(--surface); border: 1px solid var(--border); border-radius: var(--radius); padding: 13px 14px; margin-bottom: 8px; display: flex; justify-content: space-between; align-items: flex-start; }
.sale-left { flex: 1; }
.sale-name { font-weight: 600; font-size: 14px; margin-bottom: 3px; }
.sale-right { text-align: right; }
.sale-amount { font-family: var(--font-head); font-size: 16px; font-weight: 700; color: var(--accent); }
</style>