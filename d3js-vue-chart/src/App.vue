<template>
    <div class="container">
        <div class="settings">
            <label for="stocks">Stocks count</label><input type="number" id="stocks" v-model.lazy="stockNumber" min="10"
                max="10000"><span>(Max: 10000)</span>
            <label for="weeks">History</label><input type="number" id="weeks" v-model.lazy="weekNumber" min="1"
                max="1000"><span>(Max: 1000)</span>
        </div>
        <ChartVue :stocks="stocks" />
        <input type="range" min="0" :max="weekNumber-1" v-model="currentWeek"/>
        {{currentWeek}} {{typeof(currentWeek)}}
    </div>
</template>

<script setup lang="ts">
import * as d3 from "d3";
import { computed, ref, shallowRef, watch } from "vue";
import { GenerateStocks, StockItem } from "./data/stockData";
import ChartVue from "./components/Chart.vue";

const stockNumber = ref(500);
const weekNumber = ref(52 * 3);
const currentWeek = ref(0);
const allStocks = computed(()=> GenerateStocks(stockNumber.value, weekNumber.value, d3.timeDay()));
const stocks = computed(() => allStocks.value.filter(d=>d.weekOffset == currentWeek.value));

</script>

<style>
html,
body {
    margin: 0;
    padding: 0;
    font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
}

.container {
    display: flex;
    flex-direction: column;
    align-items: center;
}

.settings 
{
    max-width: 1000px;
    display: grid;
    gap: 0.2em;
    grid-template-columns: 100px 80px 100px;
}
</style>
