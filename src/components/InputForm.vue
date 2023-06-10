<template>
    <div>
        <label class="label-primary">{{ text }}</label>
        <input type="text" :value="modelValue" @input="$emit('update:modelValue', $event.target.value)" class="input-primary" />
    </div>
</template>

<script setup>

const props = defineProps({
    modelValue: {
        type: String,
        required: true
    },
    text: {
        type: String,
        default: ''
    }
});

</script>