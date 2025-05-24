<template>
    <div class="card-creator">
        <form @submit.prevent="handleSubmit" class="card-form">
            <div v-for="(field, index) in formFields" :key="index" class="form-group mb-3">
                <label :for="field.label" class="form-label">{{ field.label }}</label>
                
                <!-- Select Input -->
                <select v-if="field.type === 'select'"
                        :id="field.label"
                        v-model="formData[field.label]"
                        class="form-select"
                        :placeholder="field.placeholder">
                    <option v-for="option in field.options" 
                            :key="option.value" 
                            :value="option.value">
                        {{ option.label }}
                    </option>
                </select>

                <!-- Text Input -->
                <input v-else-if="field.type === 'text'"
                       :id="field.label"
                       v-model="formData[field.label]"
                       type="text"
                       class="form-control"
                       :placeholder="field.placeholder">

                <!-- Textarea Input -->
                <textarea v-else-if="field.type === 'textarea'"
                         :id="field.label"
                         v-model="formData[field.label]"
                         class="form-control"
                         :placeholder="field.placeholder"
                         rows="4">
                </textarea>
            </div>

            <div class="d-flex justify-content-end gap-2">
                <button type="button" 
                        class="btn btn-secondary w-25" 
                        @click="$emit('cancel')">
                    <i class="fa-solid fa-arrow-left"></i>
                </button>
                <button type="submit" 
                        class="btn btn-primary w-75">
                    <i class="fa-solid fa-check"></i>
                </button>
            </div>
        </form>
    </div>
</template>

<script setup>
import { ref, inject } from 'vue'

const props = defineProps({
    formFields: {
        type: Array,
        required: true
    }
})

const emit = defineEmits(['submit', 'cancel'])

/** @type {Function} */
const localizeFromStrings = inject("localizeFromStrings")

const formData = ref({})

const handleSubmit = () => {
    emit('submit', formData.value)
}
</script>

<style lang="scss" scoped>
.card-creator {
    padding: 1rem;
    
    .card-form {
        max-width: 600px;
        margin: 0 auto;
    }
}
</style> 