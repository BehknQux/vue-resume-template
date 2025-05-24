<template>
  <input type="number"
         class="form-control"
         :value="modelValue"
         @input="$emit('update:modelValue', $event.target.value)"
         :placeholder="placeholder"
         :min="min"
         :max="max"
         :step="step"
         :required="required">
</template>

<script setup>
defineProps({
  modelValue: {
    type: [Number, String],
    default: ''
  },
  placeholder: {
    type: String,
    default: ''
  },
  min: {
    type: Number,
    default: undefined
  },
  max: {
    type: Number,
    default: undefined
  },
  step: {
    type: Number,
    default: 1
  },
  required: {
    type: Boolean,
    default: false
  }
})

defineEmits(['update:modelValue'])
</script> 