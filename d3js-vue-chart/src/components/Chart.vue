<template>
    <div class="container">
        <div v-if="!isFinished">Loading stock data</div>
        <div v-if="error">An error occurred while loading stock data: {{ error }}</div>
        <div v-if="isFinished" class="setting">
            <label for="stocks">Stocks count</label><input type="number" v-model="stockNumber"><span>(Total: {{
                    allRows.length
            }})</span>
        </div>
        <div id="chart"></div>
    </div>
</template>

<script setup lang="ts">
// https://raw.githubusercontent.com/plotly/datasets/master/all_stocks_5yr.csv
import * as d3 from 'd3';
import { useFetch } from '@vueuse/core';
import { computed, onMounted, onUnmounted, ref, watch } from 'vue';


const stockNumber = ref(10);
const { isFinished, error, data, execute, canAbort, abort } = useFetch("https://raw.githubusercontent.com/plotly/datasets/master/all_stocks_5yr.csv", { immediate: false })

const allRows = computed(() => d3.csvParse(data.value as string))
//const stockWithPrices = computed(() => d3.group(allRows.value, (d)=>d.Name))
watch(allRows, (newRows, oldRows) => console.log(newRows));

onMounted(() => {
    execute();
});

onUnmounted(() => {
    if (canAbort.value)
        abort();
})

</script>
    
    

<style scoped>
.container {
    display: flex;
    flex-direction: column;
    align-items: center;
}

.setting {
    display: flex;
    gap: 1em;
}
</style>
