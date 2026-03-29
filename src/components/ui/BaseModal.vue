<template>
  <Teleport to="body">
    <Transition name="fade">
      <div v-if="modelValue" class="modal-overlay" @click.self="$emit('update:modelValue', false)">
        <div class="modal">
          <div v-if="title" class="modal-title">{{ title }}</div>
          <slot />
        </div>
      </div>
    </Transition>
  </Teleport>
</template>
<script setup>
defineProps({ modelValue: Boolean, title: String })
defineEmits(['update:modelValue'])
</script>