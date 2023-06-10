<template>
    <div
        class="file border-4 border-dashed border-transparent w-full bg-slate-700 dark:bg-slate-700 rounded-xl md:rounded-br-none p-6">
        <textarea class="text-sm w-full bg-slate-700 dark:bg-slate-700 text-white font-mono focus:outline-none"
            :placeholder="placeholder" autocomplete="off" autocorrect="off" autocapitalize="off" spellcheck="false" ref="textarea"
            :value="modelValue" @input="$emit('update:modelValue', $event.target.value), resizeTextarea()"></textarea>
    </div>
</template>

<script setup>
import { ref, watch } from 'vue';

const props = defineProps({
    modelValue: {
        type: String,
        required: true
    },
    placeholder: {
        type: String,
        default: ''
    }
});

const textarea = ref(null);

function resizeTextarea() {
    textarea.value.style.height = 'auto';
    textarea.value.style.height = `${textarea.value.scrollHeight + 2}px`;
};

watch(() => props.modelValue, () => {
    resizeTextarea();
});

</script>