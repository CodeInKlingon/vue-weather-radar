<script setup lang="ts">
import { MglMap } from "@indoorequal/vue-maplibre-gl";
import { southernOntario } from "@/lib/constants";
import YouAreHere from '@/components/you-are-here.vue';
import WeatherRadar from '@/components/weather-radar.vue';
</script>

<template>
	<!-- map-style="/styles/dark.json" -->
	<!-- map-style="https://tiles.openfreemap.org/styles/liberty" -->
	<mgl-map :center="southernOntario" :zoom="7" map-style="https://tiles.openfreemap.org/styles/liberty">
		<you-are-here />
		<weather-radar />
	</mgl-map>
</template>

<style>
body {
	margin: 0;
	padding: 0;
	width: 100%;
	height: 100vh;
}

#app {
	width: 100%;
	height: 100%;
}
</style>
