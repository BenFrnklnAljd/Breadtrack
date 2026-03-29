<template>
  <div class="page">
    <div class="page-title">Production Log</div>
    <div class="page-subtitle">Record daily batches</div>

    <div class="panel">
      <div class="panel-title">{{ editing ? '✏️ Edit Entry' : '+ Log Batch' }}</div>
      <div class="form-group">
        <label class="field-label">Product</label>
        <select class="field-input" v-model="form.productId">
          <option value="">Select product…</option>
          <option v-for="p in inv.activeProducts" :key="p.id" :value="p.id">{{ p.name }}</option>
        </select>
      </div>
      <div class="form-row">
        <div class="form-group">
          <label class="field-label">Qty Produced</label>
          <input class="field-input" type="number" v-model="form.qty" placeholder="e.g. 100" min="1" />
        </div>
        <div class="form-group">
          <label class="field-label">Date</label>
          <input class="field-input" type="date" v-model="form.date" />
        </div>
      </div>
      <div class="form-group">
        <label class="field-label">Notes (optional)</label>
        <input class="field-input" v-model="form.notes" placeholder="e.g. Morning batch" />
      </div>
      <p v-if="err" class="form-error">{{ err }}</p>
      <div class="panel-actions">
        <button class="btn btn-primary btn-sm" @click="save">{{ editing ? 'Update' : 'Log Batch' }}</button>
        <button v-if="editing" class="btn btn-ghost btn-sm" @click="cancel">Cancel</button>
      </div>
    </div>

    <div class="section-label">History ({{ inv.sortedProductions.length }})</div>

    <div v-if="inv.sortedProductions.length === 0" class="empty-state">
      <div class="empty-icon">🏭</div>
      <div class="empty-text">No batches logged yet.</div>
    </div>

    <div v-for="e in inv.sortedProductions" :key="e.id" class="batch-card">
      <div class="batch-top">
        <div>
          <div class="batch-name">{{ inv.productName(e.productId) }}</div>
          <div class="date-meta">{{ e.date }} · {{ e.staffName }}</div>
          <div v-if="e.notes" class="batch-notes">{{ e.notes }}</div>
        </div>
        <div class="batch-qty">
          <div style="font-size:10px;color:var(--muted)">QTY</div>
          <div style="font-family:var(--font-head);font-size:26px;font-weight:800;line-height:1">{{ e.qty }}</div>
        </div>
      </div>
      <div class="batch-footer">
  <span v-if="inv.isPerishable(e.productId)" :class="['tag', expiryTagClass(e.date)]">{{ expiryLabel(e.date) }}</span>
  <span v-else class="tag tag-bev">No Expiry</span>
  <div class="batch-actions">
    <button class="btn btn-xs" style="background:var(--red-dim);color:var(--red);border:none" @click="confirmDel(e)">🗑</button>
  </div>
</div>
    </div>

    <!-- Delete Modal -->
    <BaseModal v-model="showDel" title="Delete Entry?">
      <p style="font-size:14px;color:var(--text2);margin-bottom:20px">
        Remove <strong>{{ delTarget?.qty }} units of {{ inv.productName(delTarget?.productId) }}</strong>? This cannot be undone.
      </p>
      <div class="panel-actions">
        <button class="btn btn-danger btn-sm" @click="doDel">Delete</button>
        <button class="btn btn-ghost btn-sm" @click="showDel = false">Cancel</button>
      </div>
    </BaseModal>
  </div>
</template>

<script setup>
import { ref, reactive } from 'vue'
import { useInventoryStore } from '@/stores/inventory'
import { useAuthStore } from '@/stores/auth'
import { useToastStore } from '@/stores/toast'
import { today, expiryLabel, expiryTagClass, daysDiff, EXPIRY_DAYS } from '@/utils/date'
import BaseModal from '@/components/ui/BaseModal.vue'

const inv   = useInventoryStore()
const auth  = useAuthStore()
const toast = useToastStore()

const editing  = ref(null)
const err      = ref('')
const showDel  = ref(false)
const delTarget = ref(null)

const form = reactive({ productId: '', qty: '', date: today(), notes: '' })

function isBatchExpired(e) {
  if (!inv.isPerishable(e.productId)) return false
  return daysDiff(e.date) > EXPIRY_DAYS
}

function save() {
  err.value = ''
  if (!form.productId) { err.value = 'Select a product.'; return }
  if (!form.qty || parseInt(form.qty) < 1) { err.value = 'Enter a valid quantity.'; return }
  if (!form.date) { err.value = 'Select a date.'; return }
  if (editing.value) {
    inv.updateProduction(editing.value.id, { ...form })
    toast.success('Updated.')
    cancel()
  } else {
    inv.addProduction({ ...form, staffName: auth.currentUser.name })
    toast.success('Batch logged!')
    Object.assign(form, { productId: '', qty: '', date: today(), notes: '' })
  }
}
function startEdit(e) {
  editing.value = e
  Object.assign(form, { productId: e.productId, qty: e.qty, date: e.date, notes: e.notes })
  window.scrollTo({ top: 0, behavior: 'smooth' })
}
function cancel() {
  editing.value = null; err.value = ''
  Object.assign(form, { productId: '', qty: '', date: today(), notes: '' })
}
function confirmDel(e) { delTarget.value = e; showDel.value = true }
function doDel() {
  inv.deleteProduction(delTarget.value.id)
  toast.info('Deleted.')
  showDel.value = false
}
</script>

<style scoped>
.batch-card { background: var(--surface); border: 1px solid var(--border); border-radius: var(--radius); padding: 14px; margin-bottom: 8px; }
.batch-top { display: flex; justify-content: space-between; align-items: flex-start; margin-bottom: 10px; }
.batch-name { font-weight: 600; font-size: 15px; margin-bottom: 3px; }
.batch-notes { font-size: 12px; color: var(--muted); margin-top: 4px; }
.batch-qty { text-align: right; }
.batch-footer { display: flex; justify-content: space-between; align-items: center; }
.batch-actions { display: flex; gap: 6px; align-items: center; }
.locked-label { font-size: 11px; color: var(--muted); }
</style>