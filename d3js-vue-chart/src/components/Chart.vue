<template>
    <div ref="divChart" class="chart-container">
        <svg id="svgChart" ref="svgChart" :width="width" :height="height">
            <g id="xaxis" />
            <g id="yaxis" />
            <g id="marks" />
        </svg>
    </div>
</template>

<script setup lang="ts">
import * as d3 from 'd3';
import { useElementSize } from '@vueuse/core';
import { computed, ref, watch } from 'vue';
import { StockItem } from '../data/stockData'

const props = defineProps<{ stocks: StockItem[] }>();

const divChart = ref<HTMLDivElement | null>(null);
const divChartSize = useElementSize(divChart)

const svgChart = ref<SVGElement | null>(null);

const width = computed(() => Math.floor(d3.min([1000, divChartSize.width.value])!));
const height = computed(() => Math.floor(width.value * 9 / 16));
const margin = { left: 25, top: 20, right: 20, bottom: 25 };
const innerWidth = computed(() => width.value - margin.left - margin.right);
const innerHeight = computed(() => height.value - margin.top - margin.bottom);

watch([svgChart, props, divChartSize.width], () => {
    updateChart();
})

function updateChart() {
    if (!svgChart.value)
        return;
    const stocks = props.stocks || [];

    console.log(`Rendering chart ${width.value} / ${height.value} - ${stocks.length} stocks`)

    const xScale = d3.scaleLinear().domain([0, 10]).range([0, innerWidth.value]);
    const yScale = d3.scaleLinear().domain([0, 10]).range([innerHeight.value, 0]);
    const rScale = d3.scaleLinear().domain([d3.min(stocks, d => d.size) || 0, d3.max(stocks, d => d.size) || 0]).range([0, 20]);
    const colorScale = d3.scaleLinear().domain(rScale.domain()).range(["red", "green"] as any);


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

    const markPane = svg.select("g#marks").attr("transform", `translate(${margin.left}, ${margin.top})`)
    markPane
        .selectAll("circle")
        .data(stocks)
        .join("circle")
        .attr("cx", (d) => xScale(d.alphaX))
        .attr("cy", (d) => yScale(d.alphaY))
        .attr("r", (d) => rScale(d.size))
        .attr("fill", d=>colorScale(d.size))
    //console.log(data);
}

</script>
    
    

<style scoped>
.chart-container {
    display: grid;
    place-items: center;
    min-width: 100%;
}
</style>
