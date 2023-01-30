<template>
    <div class="flex h-10 p-4 items-center justify-between"> 
        <div class="flex">
            <span class="w-16 text-right text-gray-600 font-bold">{{ method }}:</span>
            <p class="px-1 text-blue-900 font-bold">
                {{ requestName }}
            </p>
            <svg v-if="loading && !error" class="text-sky-600 animate-spin h-5 w-5" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24">
                <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4"></circle>
                <path class="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"></path>
            </svg>
            <svg v-if="error" class="fill-red-600" xmlns="http://www.w3.org/2000/svg" height="20" width="20">
                <path d="M10 14.167q.375 0 .635-.25.261-.25.261-.646 0-.375-.261-.636-.26-.26-.635-.26t-.635.26q-.261.261-.261.636t.261.635q.26.261.635.261ZM9.125 11h1.75V5.896h-1.75ZM10 18.167q-1.688 0-3.177-.636-1.49-.635-2.604-1.75-1.115-1.114-1.75-2.604-.636-1.489-.636-3.177 0-1.708.636-3.187.635-1.48 1.75-2.594 1.114-1.115 2.604-1.75Q8.312 1.833 10 1.833q1.708 0 3.188.636 1.479.635 2.593 1.75 1.115 1.114 1.75 2.604.636 1.489.636 3.177t-.636 3.177q-.635 1.49-1.75 2.604-1.114 1.115-2.604 1.75-1.489.636-3.177.636Zm0-1.729q2.688 0 4.562-1.876 1.876-1.874 1.876-4.562t-1.876-4.562Q12.688 3.562 10 3.562T5.438 5.438Q3.562 7.312 3.562 10t1.876 4.562Q7.312 16.438 10 16.438ZM10 10Z"/>
            </svg>
            <svg v-if="!loading && !error" class="fill-green-600" xmlns="http://www.w3.org/2000/svg" height="20" width="20">
                <path d="m8.917 13.104 5.041-5.021-1.208-1.208-3.833 3.833L7.208 9l-1.187 1.208ZM10 18.167q-1.688 0-3.177-.636-1.49-.635-2.604-1.75-1.115-1.114-1.75-2.604-.636-1.489-.636-3.177 0-1.708.636-3.187.635-1.48 1.75-2.594 1.114-1.115 2.604-1.75Q8.312 1.833 10 1.833q1.708 0 3.188.636 1.479.635 2.593 1.75 1.115 1.114 1.75 2.594.636 1.479.636 3.187 0 1.688-.636 3.177-.635 1.49-1.75 2.604-1.114 1.115-2.593 1.75-1.48.636-3.188.636Zm0-1.729q2.688 0 4.562-1.876 1.876-1.874 1.876-4.562t-1.876-4.562Q12.688 3.562 10 3.562T5.438 5.438Q3.562 7.312 3.562 10t1.876 4.562Q7.312 16.438 10 16.438ZM10 10Z"/>
            </svg>
        </div>
        <div v-if="!loading" class="px-2">
            <button class="cursor-pointer w-24 p-1 text-sm font-bold items-center border rounded-md hover:shadow-md" :class=" error ? 'text-red-600' : 'text-green-600' " @click="showResult = !showResult">Show {{error ? 'error' : 'result'}}</button>
        </div>
    </div>
        <!-- Main modal -->
        <template v-if="showResult">
            <Teleport to="body">
            <div @click="showResult = false" class="fixed top-0 left-0 w-full h-full bg-black opacity-30" />
                <div class="fixed  left-20 right-20 top-24 h-5/6 bg-white max-w-2xl rounded-lg shadow-md">
                    <div class="flex items-start justify-between p-4 border-b rounded-t">
                        <h3 class="text-xl font-semibold text-gray-900 dark:text-white">
                            Response data for {{ requestName }}
                        </h3>
                        <button @click="showResult = false" type="button" class="text-gray-400 bg-transparent hover:bg-gray-200 hover:text-gray-900 rounded-lg text-sm p-1.5 ml-auto inline-flex items-center dark:hover:bg-gray-600 dark:hover:text-white">
                            <svg aria-hidden="true" class="w-5 h-5" fill="currentColor" viewBox="0 0 20 20" xmlns="http://www.w3.org/2000/svg"><path fill-rule="evenodd" d="M4.293 4.293a1 1 0 011.414 0L10 8.586l4.293-4.293a1 1 0 111.414 1.414L11.414 10l4.293 4.293a1 1 0 01-1.414 1.414L10 11.414l-4.293 4.293a1 1 0 01-1.414-1.414L8.586 10 4.293 5.707a1 1 0 010-1.414z" clip-rule="evenodd"></path></svg>
                        </button>
                    </div>
                    <p class="p-4 bg-white h-5/6 break-words overflow-y-auto">
                        {{ error ? error : result }}
                    </p>
                </div>
            </Teleport>

        </template>
</template>

<script setup>
import { ref } from 'vue';


const showResult = ref(false)

const props = defineProps({
    requestName: String,
    loading: {
        type: Boolean,
        default: true,
    },
    error: {
        type: String,
        default: null,
    },
    result: {
        type: Object,
        default: () => {},
    },
    method: {
        type: String,
        default: 'GET',
    },
})
</script>
