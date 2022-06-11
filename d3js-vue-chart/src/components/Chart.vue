<template>
    <div class="container">
        <div v-if="!fetchStocks.isFinished.value">Loading stock data</div>
        <div v-if="fetchStocks.error.value">An error occurred while loading stock data: {{ fetchStocks.error.value }}
        </div>
        <div v-if="fetchStocks.isFinished.value" class="setting">
            <label for="stocks">Stocks count</label><input type="number" v-model.lazy="stockNumber" min="10"
                :max="stockWithPrices?.size"><span>(Total: {{
                        stockWithPrices?.size
                }})</span>
        </div>
        <div ref="divChart" id="divChart"></div>
    </div>
</template>

<script setup lang="ts">
// https://raw.githubusercontent.com/plotly/datasets/master/all_stocks_5yr.csv
import * as d3 from 'd3';
import { useFetch, useElementSize } from '@vueuse/core';
import { computed, onMounted, onUnmounted, ref, watch } from 'vue';
import { csv, schemeGnBu } from 'd3';


const stockNumber = ref(10);
const fetchStocks = useFetch("https://raw.githubusercontent.com/plotly/datasets/master/all_stocks_5yr.csv", { immediate: false })

onMounted(() => {
    fetchStocks.execute();
});

onUnmounted(() => {
    if (fetchStocks.canAbort.value)
        fetchStocks.abort();
})

const allRows = computed(() => fetchStocks.isFinished.value && fetchStocks.data.value ? d3.csvParse(fetchStocks.data.value as string) : null)
const stockWithPrices = computed(() => allRows.value !== null ? d3.group(allRows.value, (d) => d.Name) : null)

const divChart = ref<HTMLDivElement | null>(null);
const divChartSize = useElementSize(divChart)

watch([stockNumber, stockWithPrices, divChartSize.width], () => {
    renderChart();
})



function renderChart() {
    if (!stockWithPrices.value)
        return;
    if (!divChart.value)
        return;
    const width = Math.floor(d3.min([1000, divChartSize.width.value])!);
    const height = 600;
    const margin = { left: 20, top: 20, right: 20, bottom: 20, }
    const innerWidth = width - margin.left - margin.right
    const innerHeight = height - margin.top - margin.bottom;

    console.log(`Rendering chart ${width} / ${height} - ${stockNumber.value} stocks`)

    const xScale = d3.scaleLinear().domain([0, 10]).range([0, innerWidth]);
    const yScale = d3.scaleLinear().domain([0, 10]).range([0, innerHeight]);

    const xAxis = d3.axisLeft(xScale);
    const yAxis = d3.axisBottom(yScale);

    let svg = d3
        .select(divChart.value)
        .selectAll("svg")
        .data([0])
        .join(
            enter => enter.append("svg"),
            update=> update
        )
        .attr("width", width)
        .attr("height", height)
        .style("background-color", "red")
        .call(xAxis as any)
        .call(yAxis as any)
}

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

#divChart {
    min-width: 100%;
    max-width: 100%;
    min-height: 10;
}
</style>
