<template>
    <div class="w-[600px] h-[80px] flex items-center">
        <TextInput
            type="text"
            class="block w-full mr-2"
            v-model="search"
            placeholder="Search for files and folders"
        />
    </div>
</template>
<script setup>
import { onMounted, ref } from "vue";
const search = ref("");
import TextInput from "@/Components/TextInput.vue";
</script>
