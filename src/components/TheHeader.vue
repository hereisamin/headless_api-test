<template>
    <div class="flex justify-between items-center h-20 bg-blue-200 mb-5">
        <a class="flex h-full items-center" href="https://net-ucommerce-docs-develop.azurewebsites.net" target="_blank">
            <img src="https://net-ucommerce-docs-develop.azurewebsites.net/images/ucommerce_widelogo_dark.svg" class="h-2/3 p-2" alt="logo" />
            <p class="font-bold text-blue-800">Headless API Test App</p>
        </a>
        <a class="flex h-10 w-10 text-center mr-8 justify-center items-center rounded-md shadow-sm hover:shadow-md font-bold text-blue-800 border" :href="`${props.baseUrl}/ucommerce-ui`" target="_blank">
            UI
        </a>
    </div>
</template>

<script setup>
const props = defineProps({
    baseUrl: {
        type: String,
        required: true,
    },
})
</script>
