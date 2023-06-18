<template>
    <div class="max-container w-full flex mt-8 md:mt-4">
        <div class="w-full space-y-6 mx-8">
            <h1 class="text-2xl text-center mt-4">
                Convert Data from CSV to PHP array
            </h1>
            <section class="shadow dark:shadow-xl rounded-xl bg-white dark:bg-slate-800 p-3 md:p-6">
                <div class="grid gap-6 mb-6 md:grid-cols-6">
                    <InputForm v-model="delimiter" text="Delimiter" />
                    <InputForm v-model="only" text="Only Column" />
                    <InputForm v-model="ignore" text="Ignore Column" />
                    <div>
                        <label class="label-primary">On ignore</label>
                        <select v-model="onIgnore" class="input-primary">
                            <option value="comment">Comment</option>
                            <option value="delete">Delete</option>
                        </select>
                    </div>
                    <div>
                        <label class="label-primary">On null</label>
                        <select v-model="onNull" class="input-primary">
                            <option value="keep">Keep</option>
                            <option value="delete">Delete</option>
                        </select>
                    </div>
                </div>
                <TextAreaForm v-model="csvData" placeholder="Paste your CSV data" />
            </section>
            <section class="shadow rounded-xl bg-white dark:bg-slate-800 p-3 md:p-6">
                <h2 class="text-xl" role="button" @click="showHeaders = !showHeaders">Headers with Index</h2>
                <div class="grid gap-6 mb-6 grid-cols-6 mt-6" v-if="showHeaders">
                    <div v-for="header in headersWithIndex" :key="header">{{ header }}</div>
                </div>
            </section>
            <section class="shadow dark:shadow-xl rounded-xl bg-white dark:bg-slate-800 p-3 md:p-6 w-full">
                <button @click="copyToClipboard()" class="mb-4">Copy</button>
                <div id="resultArray" class="overflow-x-auto max-w-full ">
                    <pre class="whitespace-pre-wrap"><code>{{ phpArray }}</code></pre>
                </div>
            </section>
        </div>
    </div>
</template>

<script setup>
import TextAreaForm from "./components/TextAreaForm.vue";
import InputForm from "./components/InputForm.vue";
import { ref, watch, computed } from "vue";

const csvData = ref("");
const phpArray = ref("");
const delimiter = ref("~");
const onIgnore = ref("delete");
const onNull = ref("keep");
const ignore = ref("");
const only = ref("");
const headersWithIndex = ref([]);
const showHeaders = ref(false);

watch(csvData, (value) => {
    transformCsv(value);
});

const columnsToIgnore = computed(() => {
    return getCleanNumbers(ignore.value);
});

const onlyColumns = computed(() => {
    return getCleanNumbers(only.value);
});

function getCleanNumbers(array) {
    if (!array) {
        return [];
    }

    return array.split(",")
        .filter((value, index, self) => self.indexOf(value) === index)
        .filter((value) => value !== "")
        .filter((column) => !isNaN(column))
        .map((column) => Number(column.trim()))
}

function transformCsv(data) {
    const updatedCsvData = data.replace(/"([^"]*)"/g, (match, capturedText) => {
        return capturedText.replace(/\n/g, ' ');
    });

    const lines = updatedCsvData.split("\n");
    const headers = lines[0].split(delimiter.value).map(header => header.trim().toLowerCase().replace(/ /g, "_"));

    headersWithIndex.value = headers.map((header, index) => {
        return index + ": " + header;
    });

    const temporalResult = lines.slice(1).map((line, index) => {

        const values = line.split(delimiter.value).map(header => header.trim());

        const rowValues = headers.map((header, index) => {
            if (onlyColumns.value.length > 0 && !onlyColumns.value.includes(index)) {
                return null;
            }

            if (columnsToIgnore.value.includes(index)) {
                if (onIgnore.value === "delete")
                    return null;
                else
                    return `    //"${header}" => "${values[index] ?? ""}",`;
            }

            if (!values[index]) {
                if (onNull.value === "delete")
                    return null;
                else
                    return `    "${header}" => "",`;
            }

            return `    "${header}" => "${values[index] ?? ""}",`;
        }).filter(value => value !== null);

        return `[\n${rowValues.join("\n")}\n],`;
    }).join("\n");

    phpArray.value = temporalResult;
}

function copyToClipboard() {
    if (!phpArray.value) {
        alert('No data to copy');
        return;
    }
    const el = document.createElement('textarea');
    el.value = phpArray.value;
    document.body.appendChild(el);
    el.select();
    document.execCommand('copy');
    document.body.removeChild(el);
    alert('Copied to clipboard');
}

watch(() => [delimiter.value, onIgnore.value, onNull.value, ignore.value, only.value], () => {
    transformCsv(csvData.value);
});

</script>
