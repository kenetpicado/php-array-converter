<template>
    <div class="max-container w-full flex mt-8 md:mt-4">
        <div class="w-full space-y-6 mx-8">
            <h1 class="text-2xl text-center mt-4">
                Convert Data from CSV to PHP array
            </h1>
            <section class="shadow dark:shadow-xl rounded-xl bg-white dark:bg-slate-800 p-3 md:p-6">

                <div class="grid gap-6 mb-6 md:grid-cols-6">
                    <InputForm v-model="delimiter" text="Delimiter" />
                    <div>
                        <label class="label-primary">Quotes (No working)</label>
                        <select v-model="quotes" class="input-primary">
                            <option value="simple">Simple</option>
                            <option value="doble">Doble</option>
                        </select>
                    </div>
                </div>

                <TextAreaForm v-model="csvData" placeholder="Paste your CSV data" />

            </section>

            <section class="shadow dark:shadow-xl rounded-xl bg-white dark:bg-slate-800 p-3 md:p-6">
                <div class="flex justify-end items-center">
                    <button @click="copyToClipboard()">Copy</button>
                </div>
                <div id="resultArray">
                    <pre>{{ phpArray }}</pre>
                </div>
            </section>
        </div>
    </div>
</template>

<script setup>
import TextAreaForm from "./components/TextAreaForm.vue";
import InputForm from "./components/InputForm.vue";
import { ref, watch } from "vue";

const csvData = ref("");
const phpArray = ref("");
const delimiter = ref(",");
const quotes = ref("simple");

watch(csvData, (value) => {
    transformCsv(value);
});

function transformCsv(data) {
    const updatedCsvData = data.replace(/"([^"]*)"/g, (match, capturedText) => {
        return capturedText.replace(/\n/g, ' ');
    });

    const lines = updatedCsvData.split("\n");
    const headers = lines[0].split(delimiter.value).map(header => header.trim());

    const temporalResult = lines.slice(1).map((line) => {
        const values = line.split(delimiter.value).map(header => header.trim());

        const rowValues = headers.map((header, index) => `    '${header}' => '${values[index] ?? ""}',`);

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

</script>
