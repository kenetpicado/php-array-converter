<script setup>
import { ref, watch } from 'vue'

const csv = ref('')
const phpArray = ref('')

watch(csv, (value) => {
  const lines = value.split('\n')
  const headers = lines[0].split(',')
  const temp = ref('')

  lines.slice(1).map(line => {
    const values = line.split(',')
    const oneRow = ref('[\n')

    headers.forEach((header, index) => {
      oneRow.value += `    '${header ?? ''}' => '${values[index] ?? ''}',\n`
    })

    oneRow.value += '],\n'
    temp.value += oneRow.value
  })

  phpArray.value = temp.value
})

</script>

<template>
  <div class="max-container w-full flex mt-8 md:mt-4">
    <div class="w-full space-y-6 mx-8">
      <h1 class="text-2xl text-center mt-4">Convert Data from CSV to PHP array</h1>
      <section class="shadow dark:shadow-xl rounded-xl bg-white dark:bg-slate-800 p-3 md:p-6">
          <div class="file border-4 border-dashed border-transparent w-full bg-slate-700 dark:bg-slate-700 rounded-xl md:rounded-br-none p-6">
            <textarea class="text-sm w-full bg-slate-700 dark:bg-slate-700 text-white font-mono focus:outline-none"
              rows="8" placeholder="Paste your CSV data" v-model="csv"
              autocomplete="off" autocorrect="off" autocapitalize="off" spellcheck="false"></textarea>
          </div>
      </section>
      <section class="shadow dark:shadow-xl rounded-xl bg-white dark:bg-slate-800 p-3 md:p-6">
        <div class="file border-4 border-dashed border-transparent w-full bg-slate-700 dark:bg-slate-700 rounded-xl md:rounded-br-none p-6">
            <textarea class="text-sm w-full bg-slate-700 dark:bg-slate-700 text-white font-mono focus:outline-none"
              rows="20" placeholder="Result" v-model="phpArray"
              autocomplete="off" autocorrect="off" autocapitalize="off" spellcheck="false"></textarea>
          </div>
      </section>
    </div>
  </div>
</template>
