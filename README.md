<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Exol | Global Grey Line</title>
    
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/leaflet/1.9.4/leaflet.css" />
    
    <style>
        :root { --exol-yellow: #ffff00; }
        body, html { margin: 0; padding: 0; height: 100%; width: 100%; background: #000; overflow: hidden; font-family: sans-serif; }
        
        /* Fixed sizing to ensure visibility */
        #map { height: 100vh; width: 100vw; background: #000; }

        .dashboard {
            position: absolute; top: 15px; right: 15px; z-index: 9999;
            background: rgba(10, 15, 20, 0.95); color: var(--exol-yellow);
            padding: 15px; border-radius: 4px; border-right: 4px solid var(--exol-yellow);
            width: 240px; box-shadow: 0 0 20px rgba(0,0,0,1);
        }
        .stat-row { display: flex; justify-content: space-between; margin: 8px 0; font-size: 0.8em; border-bottom: 1px solid #222; }
        .stat-label { color: #888; text-transform: uppercase; }
        .stat-value { font-family: monospace; font-weight: bold; }
        h2 { margin: 0; font-size: 1.1em; letter-spacing: 1px; color: var(--exol-yellow); }
        .asset-marker { background: var(--exol-yellow); width: 10px; height: 10px; border-radius: 50%; border: 2px solid #fff; box-shadow: 0 0 10px var(--exol-yellow); }
    </style>
</head>
<body>

<div class="dashboard">
    <h2>EXOL MONITOR</h2>
    <div class="stat-row"><span class="stat-label">UTC</span> <span class="stat-value" id="clock">00:00:00</span></div>
    <div class="stat-row"><span class="stat-label">Kp-Index</span> <span class="stat-value" id="kp-val">...</span></div>
    <div class="stat-row"><span class="stat-label">X-Ray</span> <span class="stat-value" id="xray-val">...</span></div>
</div>

<div id="map"></div>

<script src="https://cdnjs.cloudflare.com/ajax/libs/leaflet/1.9.4/leaflet.js"></script>
<script src="https://cdn.jsdelivr.net/npm/@joergdietrich/leaflet.terminator@1.3.0/L.Terminator.min.js"></script>

<script>
    document.addEventListener('DOMContentLoaded', function() {
        // 1. Initialize the map
        const map = L.map('map', {
            center: [20, 0],
            zoom: 2,
            minZoom: 2,
            worldCopyJump: true,
            zoomControl: false
        });

        // 2. Add Dark Matter Tiles
        L.tileLayer('https://{s}.basemaps.cartocdn.com/dark_all/{z}/{x}/{y}{r}.png', {
            attribution: '&copy; CARTO'
        }).addTo(map);

        // 3. Add Facilities
        const assets = [{n: "Atlanta", c:[33.7, -84.3]}, {n: "Bristol", c:[43.9, -69.5]}];
        assets.forEach(a => {
            L.marker(a.c, { icon: L.divIcon({className: 'asset-marker'}) }).addTo(map).bindTooltip(
