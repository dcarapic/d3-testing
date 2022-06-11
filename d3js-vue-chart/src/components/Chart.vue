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
        <div id="divChart" ref="divChart">
            <svg id="svgChart" ref="svgChart" :width="width" :height="height">
                <g id="xaxis" />
                <g id="yaxis" />
                <g id="marks" />
            </svg>
        </div>
    </div>
</template>

<script setup lang="ts">
// https://raw.githubusercontent.com/plotly/datasets/master/all_stocks_5yr.csv
import * as d3 from 'd3';
import { useFetch, useElementSize } from '@vueuse/core';
import { computed, onMounted, onUnmounted, ref, watch } from 'vue';

type Stock = { name: string, values: Array<{ date: Date, price: number }> };

const stockNumber = ref(10);
const fetchStocks = useFetch("https://raw.githubusercontent.com/plotly/datasets/master/all_stocks_5yr.csv", { immediate: false })

onMounted(() => {
    fetchStocks.execute();
});

onUnmounted(() => {
    if (fetchStocks.canAbort.value)
        fetchStocks.abort();
})

const stockWithPrices = computed(() => {
    if (!fetchStocks.isFinished.value || fetchStocks.data.value == null || fetchStocks.error.value)
        return;
    const allRows = d3.csvParse(fetchStocks.data.value as string)
    let groups = [...d3.group(allRows, d => d.Name).entries()];
    //console.log(groups);
    return groups.map((d) => ({
        name: d[0] as string,
        values: d[1].map((r) => ({
            date: new Date(r["date"] as string),
            price: parseFloat(r["open"] as string)
        }))
    }));
});

const divChart = ref<HTMLDivElement | null>(null);
const svgChart = ref<SVGElement | null>(null);
const divChartSize = useElementSize(divChart)

const width = computed(() => Math.floor(d3.min([1000, divChartSize.width.value])!));
const height = computed(() => 600);
const margin = { left: 25, top: 20, right: 20, bottom: 25 };
const innerWidth = computed(() => width.value - margin.left - margin.right);
const innerHeight = computed(() => height.value - margin.top - margin.bottom);

watch([svgChart, stockNumber, stockWithPrices, divChartSize.width], () => {
    updateChart();
})



function updateChart() {
    if (!stockWithPrices.value)
        return;
    if (!svgChart.value)
        return;

    console.log(`Rendering chart ${width.value} / ${height.value} - ${stockNumber.value} stocks`)

    const xScale = d3.scaleLinear().domain([0, 10]).range([0, innerWidth.value]);
    const yScale = d3.scaleLinear().domain([0, 10]).range([innerHeight.value, 0]);

    const maxPrice = d3.max(stockWithPrices.value, (d) => d.values[d.values.length-1].price) ?? 10;
    const xValue = d3.scaleLinear().domain([0, maxPrice]).range([0, 10])
    const yValue = d3.scaleLinear().domain([0, maxPrice]).range([0, 10])

    const xAxis = (g: d3.Selection<SVGGElement, unknown, null, undefined>) =>
        g
            .attr("transform", `translate(${margin.left}, ${height.value - margin.bottom})`)
            .call(d3.axisBottom(xScale));
    const yAxis = (g: d3.Selection<SVGGElement, unknown, null, undefined>) =>
        g
            .attr("transform", `translate(${margin.left}, ${margin.top})`)
            .call(d3.axisLeft(yScale));

    const svg = d3
        .select(svgChart.value!)
        .attr("width", width.value)
        .attr("height", height.value)
    svg.select("g#xaxis").call(xAxis as any)
    svg.select("g#yaxis").call(yAxis as any)
    const data = stockWithPrices.value.slice(0, stockNumber.value);

    const markPane = svg.select("g#marks").attr("transform", `translate(${margin.left}, ${margin.top})`)
    markPane
        .selectAll("circle")
        .data(data)
        .join("circle")
        .attr("cx", (d) => xScale(xValue(d.values[d.values.length-1].price)))
        .attr("cy", (d) => xScale(xValue(d.values[d.values.length-1].price)))
        .attr("r", 5)
    //console.log(data);
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

#svgChart {
    border: 1px solid red;
}
</style>
