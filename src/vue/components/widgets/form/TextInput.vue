<template>
  <input type="text"
         class="form-control"
         :value="modelValue"
         @input="$emit('update:modelValue', $event.target.value)"
         :placeholder="placeholder"
         :required="required">
</template>

<script setup>
defineProps({
  modelValue: {
    type: String,
    default: ''
  },
  placeholder: {
    type: String,
    default: ''
  },
  required: {
    type: Boolean,
    default: false
  }
})

defineEmits(['update:modelValue'])
</script> 