<script setup lang="ts">
import { MglRasterSource, MglRasterLayer, useMap } from '@indoorequal/vue-maplibre-gl';
import { } from '@indoorequal/vue-maplibre-gl';

import { ref, onMounted, onUnmounted } from 'vue';

const mapRef = useMap();
const radarVisible = ref(true);
const lastUpdated = ref<Date | null>(null);

// Southern Ontario radar sites - focusing on main coverage area
// const radarSources = [
// 	{
// 		id: 'radar-king',
// 		name: 'King City (WKR)',
// 		url: 'https://weather.gc.ca/radar/image_e.html?id=WKR'
// 	},
// 	{
// 		id: 'radar-exeter',
// 		name: 'Exeter (WUJ)',
// 		url: 'https://weather.gc.ca/radar/image_e.html?id=WUJ'
// 	}
// ];

// Environment Canada radar tile service
const getRadarTileUrl = () => {
	// Remove TIME parameter to get the latest available radar data
	return `https://geo.weather.gc.ca/geomet/weather/ows?SERVICE=WMS&VERSION=1.3.0&REQUEST=GetMap&BBOX={bbox-epsg-3857}&CRS=EPSG:3857&WIDTH=256&HEIGHT=256&LAYERS=RADAR_1KM_RRAI&STYLES=&FORMAT=image/png&TRANSPARENT=true`;
};

const toggleRadar = () => {
	radarVisible.value = !radarVisible.value;
};

const refreshRadar = () => {
	lastUpdated.value = new Date();
	// Force refresh by adding a cache-busting parameter
	const timestamp = Date.now();
	const refreshedUrl = `${getRadarTileUrl()}&_t=${timestamp}`;

	// Update the tiles array to force a refresh
	if (mapRef.map) {
		const source = mapRef.map.getSource('weather-radar');
		if (source && 'setTiles' in source) {
			// eslint-disable-next-line @typescript-eslint/no-explicit-any
			(source as any).setTiles([refreshedUrl]);
		}
	}
};

// Auto-refresh every 5 minutes
let refreshInterval: ReturnType<typeof setInterval>;

onMounted(() => {
	lastUpdated.value = new Date();
	refreshInterval = setInterval(refreshRadar, 5 * 60 * 1000);
});

onUnmounted(() => {
	if (refreshInterval) {
		clearInterval(refreshInterval);
	}
});
</script>

<template>
	<div>
		<!-- Radar Controls -->
		<div class="radar-controls">
			<button @click="toggleRadar" class="radar-toggle">
				{{ radarVisible ? '🌧️ Hide Radar' : '🌧️ Show Radar' }}
			</button>
			<button @click="refreshRadar" class="radar-refresh">
				🔄 Refresh
			</button>
			<div class="radar-info" v-if="lastUpdated">
				Last updated: {{ lastUpdated.toLocaleTimeString() }}
			</div>
		</div>

		<!-- Weather Radar Source and Layer -->
		<mgl-raster-source v-if="radarVisible" source-id="weather-radar" :tiles="[getRadarTileUrl()]" :tile-size="256"
			attribution="© Environment and Climate Change Canada">
			<mgl-raster-layer layer-id="weather-radar-layer" :paint="{ 'raster-opacity': 0.7 }"
				:raster-fade-duration="300" before="road_motorway_link_casing" />
		</mgl-raster-source>

		<!-- Radar Legend -->
		<div v-if="radarVisible" class="radar-legend">
			<h4>Precipitation Intensity</h4>
			<div class="legend-items">
				<div class="legend-item">
					<div class="legend-color" style="background: #00ff00"></div>
					<span>Light</span>
				</div>
				<div class="legend-item">
					<div class="legend-color" style="background: #ffff00"></div>
					<span>Moderate</span>
				</div>
				<div class="legend-item">
					<div class="legend-color" style="background: #ff8000"></div>
					<span>Heavy</span>
				</div>
				<div class="legend-item">
					<div class="legend-color" style="background: #ff0000"></div>
					<span>Very Heavy</span>
				</div>
				<div class="legend-item">
					<div class="legend-color" style="background: #ff00ff"></div>
					<span>Extreme</span>
				</div>
			</div>
		</div>
	</div>
</template>

<style scoped>
.radar-controls {
	position: absolute;
	top: 10px;
	right: 10px;
	z-index: 1000;
	display: flex;
	flex-direction: column;
	gap: 8px;
	background: rgba(255, 255, 255, 0.95);
	padding: 12px;
	border-radius: 8px;
	box-shadow: 0 2px 8px rgba(0, 0, 0, 0.15);
	backdrop-filter: blur(4px);
}

.radar-toggle,
.radar-refresh,
.radar-debug {
	padding: 8px 12px;
	border: none;
	border-radius: 6px;
	background: #2563eb;
	color: white;
	cursor: pointer;
	font-size: 14px;
	font-weight: 500;
	transition: all 0.2s ease;
}

.radar-toggle:hover,
.radar-refresh:hover,
.radar-debug:hover {
	background: #1d4ed8;
	transform: translateY(-1px);
}

.radar-debug {
	background: #dc2626;
}

.radar-debug:hover {
	background: #b91c1c;
}

.radar-info {
	font-size: 12px;
	color: #6b7280;
	text-align: center;
}

.radar-legend {
	position: absolute;
	bottom: 20px;
	left: 20px;
	z-index: 1000;
	background: rgba(255, 255, 255, 0.95);
	padding: 16px;
	border-radius: 8px;
	box-shadow: 0 2px 8px rgba(0, 0, 0, 0.15);
	backdrop-filter: blur(4px);
	min-width: 180px;
}

.radar-legend h4 {
	margin: 0 0 12px 0;
	font-size: 14px;
	font-weight: 600;
	color: #374151;
}

.legend-items {
	display: flex;
	flex-direction: column;
	gap: 6px;
}

.legend-item {
	display: flex;
	align-items: center;
	gap: 8px;
}

.legend-color {
	width: 20px;
	height: 12px;
	border-radius: 2px;
	border: 1px solid rgba(0, 0, 0, 0.1);
}

.legend-item span {
	font-size: 12px;
	color: #4b5563;
}
</style>