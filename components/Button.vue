<script setup>
    const props = defineProps({
        tag: {
            type: String,
            default: 'a'
        },
        text: {
            type: String,
            default: 'text'
        },
        link: {
            type: String
        },
        disabled: {
            type: Boolean
        }
    })
</script>

<template>
    <component 
        :is="tag.toLowerCase()" 
        :href="link" 
        :disabled="disabled"
        class="px-4 py-2 bg-indigo-500 text-white rounded-md hover:bg-indigo-600 focus:outline-none focus:ring-2 focus:ring-indigo-400 focus:ring-opacity-75 disabled:opacity-75"
    >
        <p>{{ text }}</p>
    </component>
</template>

