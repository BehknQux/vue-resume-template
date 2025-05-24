<template>
  <div class="form-group mb-3">
    <label class="form-label">{{ label }}</label>
    <component :is="inputComponent"
              :modelValue="modelValue"
              v-bind="componentProps"
              @update:modelValue="$emit('update:modelValue', $event)"/>
  </div>
</template>

<script setup>
import { computed } from 'vue'
import TextInput from './TextInput.vue'
import TextareaInput from './TextareaInput.vue'
import NumberInput from './NumberInput.vue'
import SelectInput from './SelectInput.vue'

const props = defineProps({
  type: {
    type: String,
    required: true
  },
  label: {
    type: String,
    required: true
  },
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

const inputComponent = computed(() => {
  switch (props.type) {
    case 'text':
      return TextInput
    case 'textarea':
      return TextareaInput
    case 'number':
      return NumberInput
    case 'select':
      return SelectInput
    default:
      return TextInput
  }
})

const componentProps = computed(() => {
  const baseProps = {
    placeholder: props.placeholder,
    required: props.required
  }

  if (props.type === 'select') {
    return {
      ...baseProps,
      options: props.options
    }
  }

  return baseProps
})
</script>

<style lang="scss" scoped>
.form-label {
  font-weight: 500;
  margin-bottom: 0.5rem;
}
</style> 