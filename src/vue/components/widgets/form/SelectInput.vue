<template>
  <select class="form-select"
          :value="modelValue"
          @change="$emit('update:modelValue', $event.target.value)"
          :required="required">
    <option v-if="placeholder" value="" disabled>{{ placeholder }}</option>
    <option v-for="option in options"
            :key="option.value"
            :value="option.value">
      {{ option.label }}
    </option>
  </select>
</template>

<script setup>
defineProps({
  modelValue: {
    type: [String, Number],
    default: ''
  },
  placeholder: {
    type: String,
    default: ''
  },
  options: {
    type: Array,
    default: () => []
  },
  required: {
    type: Boolean,
    default: false
  }
})

defineEmits(['update:modelValue'])
</script> 