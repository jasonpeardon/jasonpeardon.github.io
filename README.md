<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Exol | Global Grey Line Monitor</title>
    
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/leaflet@1.9.4/dist/leaflet.css" />
    
    <style>
        :root { --exol-yellow: #ffff00; --panel-bg: rgba(10, 15, 20, 0.95); }
        body, html { margin: 0; padding: 0; height: 100%; background: #000; font-family: sans-serif; overflow: hidden; }
        #map { height: 100vh; width: 100%; z-index: 1; }

        .dashboard {
            position: absolute; top: 20px; right: 20px; z-index: 2000;
            background: var(--panel-bg); color: var(--exol-yellow);
            padding: 15px; border-radius: 4px; border-right: 4px solid var(--exol-yellow);
            width: 260px; box-shadow: 0 0 20px rgba(0,0,0,1);
        }
        .stat-row { display: flex; justify-content: space-between; margin: 8px 0; font-size: 0.85em; }
        .stat-label { color: #888; text-transform: uppercase; }
        .stat-value { font-family: 'Courier New', monospace; font-weight: bold; }
        h2 { margin: 0; font-size: 1.1em; letter-spacing: 1px; }
        .asset-marker { background: var(--exol-yellow); width: 10px; height: 10px; border-radius: 50%; border: 2px solid #fff; box-shadow: 0 0 10px var(--exol-yellow); }
    </style>
</head>
<body>

<div class="dashboard">
    <h2>EXOL OPS MONITOR</h2>
    <div style="font-size: 0.6em; color: #555; margin-bottom: 10px;">REBRANDING PHASE: ACTIVE</div>
    <div class="stat-row"><span class="stat-label">UTC</span> <span class="stat-value" id="clock">--:--:--</span></div>
    <div class="stat-row"><span class="stat-label">Kp-Index</span> <span class="stat-value" id="kp-val">LOAD</span></div>
    <div class="stat-row"><span class="stat-label">X-Ray (GOES)</span> <span class="stat-value" id="xray-val">LOAD</span></div>
</div>

<div id="map"></div>

<script src="https://cdn.jsdelivr.net/npm/leaflet@1.9.4/dist/leaflet.js"></script>
<script src="https://cdn.jsdelivr.net/npm/@joergdietrich/leaflet.terminator@1.3.0/L.Terminator.min.js"></script>

<script>
    let map, terminatorLayer, propZone;

    function init() {
        map = L.map('map', { center: [20, 0], zoom: 2, worldCopyJump: true, minZoom: 2 });

        L.tileLayer('https://{s}.basemaps.cartocdn.com/dark_all/{z}/{x}/{y}{r}.png', {
            attribution: '© CARTO'
        }).addTo(map);

        // Site Overlays
        [{n: "Atlanta", c:[33.7, -84.3]}, {n: "Bristol", c:[43.9, -69.5]}].forEach(s => {
            L.marker(s.c, { icon: L.divIcon({className: 'asset-marker'}) }).addTo(map).bindTooltip(s.n);
        });

        updateTimeAndMap();
        fetchSolarData();
        setInterval(updateTimeAndMap, 1000);
        setInterval(fetchSolarData, 300000);
    }

    function updateTimeAndMap() {
        const now = new Date();
        document.getElementById('clock').innerText = now.toISOString().substr(11, 8);
        if (terminatorLayer) map.removeLayer(terminatorLayer);
        if (propZone) map.removeLayer(propZone);
        terminatorLayer = L.terminator({ fillColor: '#000', fillOpacity: 0.5, color: '#ffff00', weight: 1.5 }).addTo(map);
        propZone = L.terminator({ offset: 7.5, fillColor: '#ffff00', fillOpacity: 0.05, color: 'transparent' }).addTo(map);
    }

    // Using JSONP to bypass CORS completely
    function fetchSolarData() {
        const kpUrl = "https://services.swpc.noaa.gov/products/noaa-planetary-k-index.json";
        const xrUrl = "https://services.swpc.noaa.gov/json/goes/primary/xrays-6-hour.json";

        // Fetch Kp-Index
        const kpScript = document.createElement('script');
        kpScript.src = `https://api.allorigins.win/get?url=${encodeURIComponent(kpUrl)}&callback=handleKp`;
        document.body.appendChild(kpScript);

        // Fetch X-Ray
        const xrScript = document.createElement('script');
        xrScript.src = `https://api.allorigins.win/get?url=${encodeURIComponent(xrUrl)}&callback=handleXr`;
        document.body.appendChild(xrScript);
    }

    // Callback handlers for JSONP
    window.handleKp = function(data) {
        try {
            const raw = JSON.parse(data.contents);
            const val = raw[raw.length - 1][1];
            document.getElementById('kp-val').innerText = val;
            document.getElementById('kp-val').style.color = val >= 5 ? '#ff4444' : '#ffff00';
        } catch(e) { console.error("Kp Parse Error"); }
    };

    window.handleXr = function(data) {
        try {
            const raw = JSON.parse(data.contents);
            const longFlux = raw.filter(d => d.energy === '0.1-0.8nm');
            const val = longFlux[longFlux.length - 1].flux;
            document.getElementById('xray-val').innerText = val.toExponential(1);
        } catch(e) { console.error("Xr Parse Error"); }
    };

    window.onload = init;
</script>
</body>
</html>
