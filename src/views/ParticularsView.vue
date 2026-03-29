<template>
  <div class="page">
    <div class="page-title">Products</div>
    <div class="page-subtitle">Manage items & pricing</div>
    <div class="panel">
      <div class="panel-title">{{ editing ? '✏️ Edit Product' : '+ New Product' }}</div>
      <div class="form-group">
        <label class="field-label">Product Name</label>
        <input class="field-input" v-model="form.name" placeholder="e.g. Pandesal, Espresso" />
      </div>
      <div class="form-row">
        <div class="form-group"><label class="field-label">Category</label><select class="field-input" v-model="form.category"><option value="food">🍞 Food</option><option value="beverage">☕ Beverage</option></select></div>
        <div class="form-group"><label class="field-label">Price (₱)</label><input class="field-input" type="number" v-model="form.price" placeholder="0.00" min="0" /></div>
      </div>
      <p v-if="err" class="form-error">{{ err }}</p>
      <div class="panel-actions">
        <button class="btn btn-primary btn-sm" @click="save">{{ editing ? 'Update' : 'Add Product' }}</button>
        <button v-if="editing" class="btn btn-ghost btn-sm" @click="cancel">Cancel</button>
      </div>
    </div>
    <div class="section-label">All Products ({{ inv.products.length }})</div>
    <div v-if="inv.products.length === 0" class="empty-state"><div class="empty-icon">🏷️</div><div class="empty-text">No products yet.</div></div>
    <div v-for="p in inv.products" :key="p.id" class="product-card" :class="{ inactive: !p.active }">
      <div class="product-top">
        <div>
          <div class="product-name">{{ p.name }}</div>
          <div class="product-meta">
            <span :class="p.category === 'food' ? 'tag tag-food' : 'tag tag-bev'">{{ p.category }}</span>
            <span class="price-tag">₱{{ p.price }}</span>
            <span v-if="!p.active" class="tag tag-muted">Inactive</span>
          </div>
        </div>
        <div class="product-bal">
          <div style="font-size:10px;color:var(--muted);text-transform:uppercase;letter-spacing:.06em">Balance</div>
          <div style="font-family:var(--font-head);font-size:18px;font-weight:800" :class="inv.isExpired(p.id) ? 'text-red' : inv.currentBalance(p.id) > 0 ? 'text-green' : 'text-muted'">{{ inv.currentBalance(p.id) }}</div>
        </div>
      </div>
      <div class="product-actions">
        <button class="btn btn-ghost btn-xs" @click="startEdit(p)">✏️ Edit</button>
        <button class="btn btn-ghost btn-xs" @click="toggle(p)">{{ p.active ? '🚫 Deactivate' : '✅ Activate' }}</button>
      </div>
    </div>
  </div>
</template>
<script setup>
import { ref, reactive } from 'vue'
import { useInventoryStore } from '@/stores/inventory'
import { useToastStore } from '@/stores/toast'
const inv = useInventoryStore(); const toast = useToastStore()
const editing = ref(null); const err = ref('')
const form = reactive({ name: '', category: 'food', price: '' })
function save() { err.value = ''; if (!form.name.trim()) { err.value = 'Name required.'; return }; if (!form.price || parseFloat(form.price) <= 0) { err.value = 'Enter valid price.'; return }; if (editing.value) { inv.updateProduct(editing.value.id, { ...form }); toast.success('Updated.'); cancel() } else { inv.addProduct({ ...form }); toast.success('Added!'); Object.assign(form, { name: '', category: 'food', price: '' }) } }
function startEdit(p) { editing.value = p; Object.assign(form, { name: p.name, category: p.category, price: p.price }); window.scrollTo({ top: 0, behavior: 'smooth' }) }
function cancel() { editing.value = null; err.value = ''; Object.assign(form, { name: '', category: 'food', price: '' }) }
function toggle(p) { inv.toggleProductActive(p.id); toast.info(p.active ? 'Deactivated.' : 'Activated.') }
</script>
<style scoped>
.product-card { background: var(--surface); border: 1px solid var(--border); border-radius: var(--radius); padding: 14px; margin-bottom: 8px; transition: opacity .2s; }
.product-card.inactive { opacity: .5; }
.product-top { display: flex; justify-content: space-between; align-items: flex-start; margin-bottom: 10px; }
.product-name { font-weight: 600; font-size: 15px; margin-bottom: 6px; }
.product-meta { display: flex; align-items: center; gap: 8px; flex-wrap: wrap; }
.product-bal { text-align: right; }
.product-actions { display: flex; gap: 8px; }
</style>
