<template>
  <div class="stat-card">
    <div class="stat-label">{{ label }}</div>
    <div class="stat-val" :class="color">{{ value }}</div>
  </div>
</template>
<script setup>
defineProps({ label: String, value: [String, Number], color: { type: String, default: '' } })
</script>