<template>
    <AuthenticatedLayout>
        <nav class="flex items-center justify-between p-1 mb-3">
            <ol class="inline-flex items-center space-x-1 md:space-x-3">
                <li
                    v-for="ans of ancestors.data"
                    :key="ans.id"
                    class="inline-flex items-center"
                >
                    <Link
                        v-if="!ans.parent_id"
                        :href="route('myFiles')"
                        class="inline-flex items-center text-sm font-medium text-gray-700 hover:text-blue-600 dark:text-gray-400 dark:hover:text-white"
                    >
                        <HomeIcon class="w-4 h-4" />
                        My Files
                    </Link>
                    <div v-else class="flex items-center">
                        <svg
                            aria-hidden="true"
                            class="w-6 h-6 text-gray-400"
                            fill="currentColor"
                            viewBox="0 0 20 20"
                            xmlns="http://www.w3.org/2000/svg"
                        >
                            <path
                                fill-rule="evenodd"
                                d="M7.293 14.707a1 1 0 010-1.414L10.586 10 7.293 6.707a1 1 0 011.414-1.414l4 4a1 1 0 010 1.414l-4 4a1 1 0 01-1.414 0z"
                                clip-rule="evenodd"
                            ></path>
                        </svg>
                        <Link
                            :href="route('myFiles', { folder: ans.path })"
                            class="ml-1 text-sm font-medium text-gray-700 hover:text-blue-600 md:ml-2 dark:text-gray-400 dark:hover:text-white"
                        >
                            {{ ans.name }}
                        </Link>
                    </div>
                </li>
            </ol>
            <div>
                <DeleteFilesButton
                    :delete-all="allSelected"
                    :delete-id="selectedIds"
                />
            </div>
        </nav>
        <div class="flex-1 overflow-auto">
            <table class="min-w-full">
                <thead class="bg-gray-100 border-b">
                    <tr>
                        <th
                            class="text-sm font-medium text-gray-900 px-6 py-4 text-left"
                        >
                            <Checkbox
                                @change="onSelectAllChange"
                                v-model:checked="allSelected"
                            ></Checkbox>
                        </th>
                        <th
                            class="text-sm font-medium text-gray-900 px-6 py-4 text-left"
                        >
                            Name
                        </th>
                        <th
                            class="text-sm font-medium text-gray-900 px-6 py-4 text-left"
                        >
                            Path
                        </th>
                        <th
                            class="text-sm font-medium text-gray-900 px-6 py-4 text-left"
                        >
                            Owner
                        </th>
                        <th
                            class="text-sm font-medium text-gray-900 px-6 py-4 text-left"
                        >
                            Last Modified
                        </th>
                        <th
                            class="text-sm font-medium text-gray-900 px-6 py-4 text-left"
                        >
                            Size
                        </th>
                    </tr>
                </thead>
                <tbody>
                    <tr
                        v-for="file of allFiles.data"
                        :key="file.id"
                        @click="($event) => toggleFileSelect(file)"
                        @dblclick="openFolder(file)"
                        class="border-b transition duration-300 ease-in-out hover:bg-blue-100 cursor-pointer"
                        :class="
                            selected[file.id] || allSelected
                                ? 'bg-blue-50'
                                : 'bg-white'
                        "
                    >
                        <td
                            class="px-6 py-4 whitespace-nowrap text-sm font-medium text-gray-900 w-[30px] max-w-[30px] pr-0"
                        >
                            <Checkbox
                                @change="
                                    ($event) => onSelectCheckboxChange(file)
                                "
                                v-model="selected[file.id]"
                                :checked="selected[file.id] || allSelected"
                            ></Checkbox>
                        </td>
                        <td
                            class="px-6 py-4 whitespace-nowrap text-sm font-medium text-gray-900 flex items-center"
                        >
                            <FileIcon :file="file" />
                            {{ file.name }}
                        </td>
                        <td
                            class="px-6 py-4 whitespace-nowrap text-sm font-medium text-gray-900"
                        >
                            {{ file.path }}
                        </td>
                        <td
                            class="px-6 py-4 whitespace-nowrap text-sm font-medium text-gray-900"
                        >
                            {{ file.owner }}
                        </td>
                        <td
                            class="px-6 py-4 whitespace-nowrap text-sm font-medium text-gray-900"
                        >
                            {{ file.updated_at }}
                        </td>
                        <td
                            class="px-6 py-4 whitespace-nowrap text-sm font-medium text-gray-900"
                        >
                            {{ file.size }}
                        </td>
                    </tr>
                </tbody>
            </table>
            <div
                v-if="!allFiles.data.length"
                class="py-8 text-center text-sm text-gray-400"
            >
                There is no data in this folder
            </div>
            <div ref="loadMoreIntersect"></div>
        </div>
    </AuthenticatedLayout>
</template>
<script setup>
import AuthenticatedLayout from "@/Layouts/AuthenticatedLayout.vue";
import FileIcon from "@/Components/app/FileIcon.vue";
import { router, useForm, usePage } from "@inertiajs/vue3";
import { HomeIcon } from "@heroicons/vue/20/solid";
import { Link } from "@inertiajs/vue3";
import { httpGet } from "@/Helper/http-helper.js";
import { computed, onMounted, onUpdated, ref } from "vue";
import Checkbox from "@/Components/Checkbox.vue";
import DeleteFilesButton from "@/Components/app/DeleteFilesButton.vue";

const props = defineProps({
    files: Object,
    folder: Object,
    ancestors: Object,
});

const loadMoreIntersect = ref(null);
const allFiles = ref({
    data: props.files.data,
    next: props.files.links.next,
});
const allSelected = ref(false);
const selected = ref({});

function openFolder(file) {
    if (!file.is_folder) {
        return;
    }
    router.visit(route("myFiles", { folder: file.path }));
}

function loadMore() {
    if (allFiles.value.next == null) {
        return;
    }
    var test = allFiles.value.data;
    httpGet(allFiles.value.next).then((res) => {
        allFiles.value.data = [...allFiles.value.data, ...res.data];
        allFiles.value.next = res.links.next;
    });
}

function onSelectAllChange() {
    allFiles.value.data.forEach((f) => {
        selected.value[f.id] = allSelected.value;
    });
}

function onSelectCheckboxChange(file) {
    if (!selected.value[file.id]) {
        allSelected.value = false;
    } else {
        let checked = true;
        for (let file of allFiles.value.data) {
            if (!selected.value[file.id]) {
                checked = false;
                break;
            }
        }
        allSelected.value = checked;
    }
}

function toggleFileSelect(file) {
    selected.value[file.id] = !selected.value[file.id];
    onSelectCheckboxChange(file);
}

onMounted(() => {
    const observer = new IntersectionObserver((entries) =>
        entries.forEach((entry) => entry.isIntersecting && loadMore())
    );
    observer.observe(loadMoreIntersect.value);
});

const selectedIds = computed(() =>
    Object.entries(selected.value)
        .filter((a) => a[1])
        .map((a) => a[0])
);
</script>
