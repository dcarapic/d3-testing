<template>
    <div class="container">
        <div class="settings">
            <label for="stocks">Stocks count</label><input type="number" id="stocks" v-model.lazy="stockNumber" min="10"
                :max="StockNames.length"><span>(Max: {{ StockNames.length }})</span>
            <label for="weeks">History</label><input type="number" id="weeks" v-model.lazy="weekNumber" min="1"
                max="1000"><span>(Max: 1000)</span>
        </div>
        <ChartVue :stocks="stocks || []" />
        <div class="range">
            <button @click="animPlay">▶</button>
            <button @click="animPause">⏸</button>
            <button @click="animStop">⏹</button>
            <input type="range" min="0" :max="weekNumber - 1" style="width: 200px" v-model.number="currentWeek" />
            {{ currentWeek }}
        </div>
    </div>
</template>

<script setup lang="ts">
import * as d3 from "d3";
import { computed, ref, shallowRef, watch } from "vue";
import { GenerateStocks, StockNames } from "./data/stockData";
import ChartVue from "./components/Chart.vue";
import { useInterval } from "@vueuse/core";

const stockNumber = ref(500);
const weekNumber = ref(52 * 3);
const currentWeek = ref(0);
const allStocks = computed(() => d3.group(GenerateStocks(stockNumber.value, weekNumber.value, d3.timeDay()), d => d.weekOffset));
const stocks = computed(() => allStocks.value.get(currentWeek.value));


const playState = ref<'stop' | 'play' | 'paused'>('stop')

function animPlay() {
    console.log(`animPlay() / ${playState.value}`)
    if (playState.value !== 'play')
        playState.value = "play";
}

function animStop() {
    console.log(`animStop() / ${playState.value}`)
    if (playState.value !== 'stop') {
        playState.value = "stop";
        currentWeek.value = 0
    }
}

function animPause() {
    console.log(`animPause() / ${playState.value}`)
    if (playState.value !== 'paused')
        playState.value = "paused";
}

let animInterval = -1;
watch(playState, (newValue, oldValue) => {
    if (newValue === "play") {
        animInterval = setInterval(() => {
            if (currentWeek.value >= weekNumber.value)
                currentWeek.value = 0;
            else
                currentWeek.value += 1;
        }, 1000 / 60);
    } else if (animInterval > 0)
        clearInterval(animInterval);
})


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

.settings {
    max-width: 1000px;
    display: grid;
    gap: 0.2em;
    grid-template-columns: 100px 80px 100px;
}

.range {
    display: flex;
    gap: 0.5em
}
</style>
