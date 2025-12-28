<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>Driving Empire Analyzer</title>
  <style>
    :root {
      --bg: #0f1218;
      --panel: #161b22;
      --text: #e6edf3;
      --muted: #8b949e;
      --accent: #2f81f7;
      --good: #22c55e;   /* verde */
      --mid: #facc15;    /* amarillo */
      --meh: #fb923c;    /* naranja */
      --bad: #ef4444;    /* rojo */
      --wire: #233044;
    }
    * { box-sizing: border-box; }
    html, body {
      margin: 0; padding: 0; background: var(--bg); color: var(--text);
      font-family: system-ui, -apple-system, Segoe UI, Roboto, Ubuntu, "Helvetica Neue", Arial, "Noto Sans", "Apple Color Emoji", "Segoe UI Emoji", "Segoe UI Symbol";
    }
    header {
      padding: 12px 16px; border-bottom: 1px solid #1f2630; background: var(--panel);
      position: sticky; top: 0; z-index: 10;
    }
    .title { font-weight: 700; font-size: 18px; }
    .sub { color: var(--muted); font-size: 13px; }

    .wrap {
      display: grid;
      grid-template-columns: 1fr 320px;
      gap: 12px;
      padding: 12px;
    }
    @media (max-width: 900px) {
      .wrap { grid-template-columns: 1fr; }
    }

    .card {
      background: var(--panel);
      border: 1px solid #1f2630;
      border-radius: 10px;
      padding: 12px;
    }

    .controls {
      display: grid;
      grid-template-columns: repeat(12, 1fr);
      gap: 8px;
      align-items: center;
    }
    .controls .wide { grid-column: span 12; }
    .controls .half { grid-column: span 6; }
    .controls .third { grid-column: span 4; }
    @media (max-width: 600px) {
      .controls .half, .controls .third { grid-column: span 12; }
    }

    label {
      font-size: 12px; color: var(--muted); display: block; margin-bottom: 6px;
    }
    input[type="file"], select, button {
      width: 100%;
      padding: 10px 12px;
      background: #0b0f15;
      border: 1px solid #233044;
      border-radius: 8px;
      color: var(--text);
      font-size: 14px;
    }
    button {
      background: #0b1628;
      border: 1px solid #224b8d;
      cursor: pointer;
      transition: background .15s ease, border-color .15s ease, transform .08s ease;
    }
    button:hover { background: #0d1c36; border-color: var(--accent); }
    button:active { transform: scale(0.99); }
    .btn-danger { background: #1b0f12; border-color: #7a1d28; }
    .btn-danger:hover { background: #2a1318; border-color: #d33b4a; }

    .canvas-wrap {
      position: relative;
      aspect-ratio: 16/9;
      background: #0b0f15;
      border: 1px solid var(--wire);
      border-radius: 10px;
      overflow: hidden;
    }
    canvas {
      width: 100%;
      height: 100%;
      display: block;
      image-rendering: crisp-edges;
    }
    .legend {
      display: flex; gap: 10px; flex-wrap: wrap; margin-top: 10px;
      font-size: 12px; color: var(--muted);
    }
    .dot { width: 10px; height: 10px; border-radius: 50%; display: inline-block; margin-right: 6px; vertical-align: middle; }
    .legend-item { display: inline-flex; align-items: center; gap: 6px; padding: 4px 8px; background: #0b0f15; border: 1px solid var(--wire); border-radius: 8px; }

    .stats {
      display: grid;
      grid-template-columns: 1fr 1fr;
      gap: 8px;
      font-size: 13px;
    }
    @media (max-width: 400px) { .stats { grid-template-columns: 1fr; } }
    .stat { background: #0b0f15; border: 1px solid var(--wire); border-radius: 8px; padding: 10px; }
    .stat .head { color: var(--muted); font-size: 12px; }
    .stat .val { font-weight: 700; font-size: 16px; }

    .footer-note { color: var(--muted); font-size: 12px; margin-top: 8px; }
    .mono { font-family: ui-monospace, SFMono-Regular, Menlo, Monaco, Consolas, "Liberation Mono", "Courier New", monospace; }

    .row { display: flex; gap: 8px; }
    .row > * { flex: 1; }
  </style>
</head>
<body>
  <header>
    <div class="title">Driving Empire — Race Analyzer</div>
    <div class="sub">Carga tu JSON desde Roblox, visualizá el trazado, evaluá el desempeño y guardá múltiples laps.</div>
  </header>

  <div class="wrap">
    <main class="card">
      <div class="controls">
        <div class="third">
          <label for="fileInput">Cargar JSON</label>
          <input id="fileInput" type="file" accept="application/json" />
        </div>
        <div class="third">
          <label for="lapSelect">Seleccionar lap</label>
          <select id="lapSelect"></select>
        </div>
        <div class="third">
          <label>Acciones</label>
          <div class="row">
            <button id="btnPrev">Ver anterior</button>
            <button id="btnNext">Ver siguiente</button>
          </div>
        </div>

        <div class="third">
          <label>Persistencia</label>
          <div class="row">
            <button id="btnSave">Guardar lap</button>
            <button id="btnDelete" class="btn-danger">Borrar lap</button>
          </div>
        </div>
        <div class="third">
          <label>Vista</label>
          <div class="row">
            <button id="btnFit">Ajustar a pantalla</button>
            <button id="btnReset">Reset zoom</button>
          </div>
        </div>
        <div class="third">
          <label>Herramientas</label>
          <div class="row">
            <button id="btnExport">Exportar JSON</button>
            <button id="btnClearAll" class="btn-danger">Borrar todo</button>
          </div>
        </div>
        <div class="wide footer-note">
          Formato esperado (muestras por tiempo): [{"time": 0.0, "pos": {"x": 0, "y": 0, "z": 0}, "speed": 0, "steer": 0, "throttle": 0, "collision": false, "skid": false, "steerCorrection": 0}, ...]
        </div>
      </div>

      <div class="canvas-wrap" id="canvasWrap">
        <canvas id="trackCanvas"></canvas>
      </div>

      <div class="legend">
        <span class="legend-item"><span class="dot" style="background: var(--good)"></span> Verde: perfecto / bien</span>
        <span class="legend-item"><span class="dot" style="background: var(--mid)"></span> Amarillo: ni muy bien ni tan mal</span>
        <span class="legend-item"><span class="dot" style="background: var(--meh)"></span> Naranja: más o menos</span>
        <span class="legend-item"><span class="dot" style="background: var(--bad)"></span> Rojo: mal</span>
      </div>
    </main>

    <aside class="card">
      <div style="font-weight:700; margin-bottom:8px;">Estadísticas del lap</div>
      <div class="stats" id="stats">
        <div class="stat">
          <div class="head">Duración</div>
          <div class="val" id="statDuration">—</div>
        </div>
        <div class="stat">
          <div class="head">Distancia</div>
          <div class="val" id="statDistance">—</div>
        </div>
        <div class="stat">
          <div class="head">Colisiones</div>
          <div class="val" id="statCollisions">—</div>
        </div>
        <div class="stat">
          <div class="head">Verde / Amarillo / Naranja / Rojo</div>
          <div class="val" id="statColors">—</div>
        </div>
      </div>
      <div class="footer-note mono" id="status">Listo.</div>
    </aside>
  </div>

  <script>
    // --- Estado global ---
    const LS_KEY = 'drivingEmpireLaps';
    let laps = []; // [{id, name, samples, bounds, maxSpeed}]
    let currentLapIndex = -1;

    // --- DOM refs ---
    const fileInput = document.getElementById('fileInput');
    const lapSelect = document.getElementById('lapSelect');
    const btnPrev = document.getElementById('btnPrev');
    const btnNext = document.getElementById('btnNext');
    const btnSave = document.getElementById('btnSave');
    const btnDelete = document.getElementById('btnDelete');
    const btnFit = document.getElementById('btnFit');
    const btnReset = document.getElementById('btnReset');
    const btnExport = document.getElementById('btnExport');
    const btnClearAll = document.getElementById('btnClearAll');
    const statusEl = document.getElementById('status');

    const canvas = document.getElementById('trackCanvas');
    const wrap = document.getElementById('canvasWrap');
    const ctx = canvas.getContext('2d');

    const statDuration = document.getElementById('statDuration');
    const statDistance = document.getElementById('statDistance');
    const statCollisions = document.getElementById('statCollisions');
    const statColors = document.getElementById('statColors');

    // --- Canvas sizing / viewport ---
    let view = { scale: 1, offsetX: 0, offsetY: 0 }; // pan/zoom
    function resizeCanvas() {
      const rect = wrap.getBoundingClientRect();
      canvas.width = Math.floor(rect.width * devicePixelRatio);
      canvas.height = Math.floor(rect.height * devicePixelRatio);
      redrawAll();
    }
    window.addEventListener('resize', resizeCanvas);
    resizeCanvas();

    // --- Utilidades ---
    function fmtMs(ms) {
      const s = ms / 1000;
      return s.toFixed(2) + ' s';
    }
    function fmtDist(m) {
      return m.toFixed(1) + ' m';
    }
    function colorForScore(score) {
      if (score >= 0.8) return getCSS('--good');
      if (score >= 0.6) return getCSS('--mid');
      if (score >= 0.4) return getCSS('--meh');
      return getCSS('--bad');
    }
    function getCSS(name) {
      return getComputedStyle(document.documentElement).getPropertyValue(name).trim();
    }

    // --- Evaluación del desempeño por muestra ---
    // Heurística general:
    // - Colisión => rojo
    // - Skid fuerte o corrección de dirección alta => naranja/rojo
    // - Dirección extrema y throttle bajo => naranja
    // - Bueno cuando sin eventos, steering moderado, throttle y velocidad consistentes
    function evaluateSample(sample, context) {
      let score = 1.0;
      if (sample.collision) return 0.0;

      const maxSpeed = Math.max(1e-6, context.maxSpeed || 1.0);
      const relSpeed = Math.min(1.0, Math.max(0, sample.speed / maxSpeed));

      if (sample.skid) score -= 0.45;
      if (Math.abs(sample.steerCorrection || 0) > 0.6) score -= 0.35;
      if (Math.abs(sample.steer || 0) > 0.85) score -= 0.20;

      // Penalizar baja relación entre throttle y velocidad sostenida
      if ((sample.throttle || 0) < 0.2 && relSpeed > 0.5) score -= 0.15;

      // Suavizado según estabilidad
      score = Math.max(0, Math.min(1, score));
      return score;
    }

    // --- Carga y parsing del JSON ---
    fileInput.addEventListener('change', async (e) => {
      const file = e.target.files?.[0];
      if (!file) return;
      try {
        const text = await file.text();
        const parsed = JSON.parse(text);
        const samples = normalizeSamples(parsed);
        if (!samples || !samples.length) {
          throw new Error('JSON vacío o formato no reconocido');
        }
        const lap = buildLap(samples, file.name);
        laps.push(lap);
        currentLapIndex = laps.length - 1;
        updateLapSelector();
        redrawAll();
        report('Lap cargado: ' + lap.name);
      } catch (err) {
        console.error(err);
        report('Error al cargar JSON: ' + err.message);
      } finally {
        fileInput.value = '';
      }
    });

    function normalizeSamples(data) {
      // Soporta: array plano de samples OR {samples: [...]}
      const arr = Array.isArray(data) ? data : (Array.isArray(data.samples) ? data.samples : null);
      if (!arr) return null;
      // Aseguramos campos y orden por time
      const cleaned = arr
        .map(s => ({
          time: toNumber(s.time),
          pos: {
            x: toNumber(s.pos?.x),
            y: toNumber(s.pos?.y),
            z: toNumber(s.pos?.z)
          },
          speed: toNumber(s.speed),
          steer: toNumber(s.steer),
          throttle: toNumber(s.throttle),
          collision: !!s.collision,
          skid: !!s.skid,
          steerCorrection: toNumber(s.steerCorrection)
        }))
        .filter(s => isFinite(s.time) && isFinite(s.pos.x) && isFinite(s.pos.y));
      cleaned.sort((a, b) => a.time - b.time);
      return cleaned;
    }
    function toNumber(x) {
      const n = Number(x);
      return isFinite(n) ? n : 0;
    }

    function buildLap(samples, name = 'Lap') {
      // Bounds y maxSpeed
      let minX = Infinity, minY = Infinity, maxX = -Infinity, maxY = -Infinity, maxSpeed = 0;
      for (const s of samples) {
        if (s.pos.x < minX) minX = s.pos.x;
        if (s.pos.x > maxX) maxX = s.pos.x;
        if (s.pos.y < minY) minY = s.pos.y;
        if (s.pos.y > maxY) maxY = s.pos.y;
        if (s.speed > maxSpeed) maxSpeed = s.speed;
      }
      const bounds = { minX, minY, maxX, maxY, width: maxX - minX, height: maxY - minY };
      const id = Date.now() + '-' + Math.random().toString(36).slice(2, 7);
      return { id, name, samples, bounds, maxSpeed };
    }

    // --- LocalStorage ---
    function saveAll() {
      try {
        const data = laps.map(({ id, name, samples }) => ({ id, name, samples }));
        localStorage.setItem(LS_KEY, JSON.stringify(data));
        report('Guardado en localStorage (' + data.length + ' laps).');
      } catch (err) {
        console.error(err);
        report('Error al guardar: ' + err.message);
      }
    }
    function loadAll() {
      try {
        const raw = localStorage.getItem(LS_KEY);
        if (!raw) return;
        const data = JSON.parse(raw);
        laps = data.map(d => buildLap(d.samples, d.name));
        currentLapIndex = laps.length ? 0 : -1;
        updateLapSelector();
        redrawAll();
        report('Cargados ' + laps.length + ' laps desde localStorage.');
      } catch (err) {
        console.error(err);
        report('Error al cargar localStorage: ' + err.message);
      }
    }
    function clearAll() {
      localStorage.removeItem(LS_KEY);
      laps = [];
      currentLapIndex = -1;
      updateLapSelector();
      redrawAll();
      report('LocalStorage borrado y memoria limpia.');
    }
    loadAll();

    // --- UI acciones ---
    btnSave.addEventListener('click', () => {
      if (currentLapIndex < 0) return report('No hay lap seleccionado.');
      saveAll();
    });
    btnDelete.addEventListener('click', () => {
      if (currentLapIndex < 0) return report('No hay lap para borrar.');
      const del = laps.splice(currentLapIndex, 1)[0];
      if (laps.length === 0) currentLapIndex = -1;
      else currentLapIndex = Math.max(0, currentLapIndex - 1);
      updateLapSelector();
      redrawAll();
      report('Lap borrado: ' + del.name);
      saveAll();
    });
    btnPrev.addEventListener('click', () => {
      if (!laps.length) return report('No hay laps.');
      currentLapIndex = (currentLapIndex - 1 + laps.length) % laps.length;
      updateLapSelector();
      redrawAll();
    });
    btnNext.addEventListener('click', () => {
      if (!laps.length) return report('No hay laps.');
      currentLapIndex = (currentLapIndex + 1) % laps.length;
      updateLapSelector();
      redrawAll();
    });
    btnFit.addEventListener('click', () => {
      fitViewToLap();
      redrawAll();
    });
    btnReset.addEventListener('click', () => {
      view = { scale: 1, offsetX: 0, offsetY: 0 };
      redrawAll();
    });
    btnExport.addEventListener('click', () => {
      if (currentLapIndex < 0) return report('No hay lap para exportar.');
      const lap = laps[currentLapIndex];
      const blob = new Blob([JSON.stringify(lap.samples, null, 2)], { type: 'application/json' });
      const a = document.createElement('a');
      a.href = URL.createObjectURL(blob);
      a.download = (lap.name || 'lap') + '.json';
      a.click();
      URL.revokeObjectURL(a.href);
      report('Exportado: ' + a.download);
    });
    btnClearAll.addEventListener('click', () => {
      if (confirm('¿Borrar todos los laps y el localStorage?')) clearAll();
    });

    lapSelect.addEventListener('change', (e) => {
      const idx = Number(e.target.value);
      if (isFinite(idx)) {
        currentLapIndex = idx;
        redrawAll();
      }
    });

    function updateLapSelector() {
      lapSelect.innerHTML = '';
      if (!laps.length) {
        const opt = document.createElement('option');
        opt.value = '-1';
        opt.textContent = '— sin laps —';
        lapSelect.appendChild(opt);
        return;
      }
      laps.forEach((lap, i) => {
        const opt = document.createElement('option');
        opt.value = String(i);
        const duration = computeDuration(lap.samples);
        opt.textContent = `${i + 1}. ${lap.name} (${fmtMs(duration)})`;
        lapSelect.appendChild(opt);
      });
      lapSelect.value = String(Math.max(0, currentLapIndex));
    }

    // --- Estadísticas ---
    function computeDuration(samples) {
      if (!samples.length) return 0;
      return Math.max(0, (samples.at(-1).time - samples[0].time) * 1000);
    }
    function computeDistance(samples) {
      let dist = 0;
      for (let i = 1; i < samples.length; i++) {
        const a = samples[i - 1].pos, b = samples[i].pos;
        const dx = b.x - a.x, dy = b.y - a.y;
        dist += Math.hypot(dx, dy);
      }
      return dist;
    }
    function computeCollisions(samples) {
      let c = 0;
      for (const s of samples) if (s.collision) c++;
      return c;
    }
    function computeColorCounts(samples, context) {
      let g = 0, y = 0, n = 0, r = 0;
      for (const s of samples) {
        const score = evaluateSample(s, context);
        if (score >= 0.8) g++;
        else if (score >= 0.6) y++;
        else if (score >= 0.4) n++;
        else r++;
      }
      return { g, y, n, r };
    }

    function updateStats(lap) {
      if (!lap) {
        statDuration.textContent = '—';
        statDistance.textContent = '—';
        statCollisions.textContent = '—';
        statColors.textContent = '—';
        return;
      }
      const duration = computeDuration(lap.samples);
      const distance = computeDistance(lap.samples);
      const collisions = computeCollisions(lap.samples);
      const colors = computeColorCounts(lap.samples, { maxSpeed: lap.maxSpeed });
      statDuration.textContent = fmtMs(duration);
      statDistance.textContent = fmtDist(distance);
      statCollisions.textContent = String(collisions);
      statColors.textContent = `${colors.g} / ${colors.y} / ${colors.n} / ${colors.r}`;
    }

    // --- Dibujo en canvas ---
    function fitViewToLap() {
      if (currentLapIndex < 0) return;
      const lap = laps[currentLapIndex];
      const b = lap.bounds;
      const pad = 20; // px
      const cw = canvas.width, ch = canvas.height;

      const scaleX = (cw - 2 * pad) / (b.width || 1);
      const scaleY = (ch - 2 * pad) / (b.height || 1);
      const scale = Math.min(scaleX, scaleY);

      view.scale = scale;
      // Centro en bounds
      const centerX = (b.minX + b.maxX) / 2;
      const centerY = (b.minY + b.maxY) / 2;
      const screenCenterX = cw / 2;
      const screenCenterY = ch / 2;

      view.offsetX = screenCenterX - centerX * scale;
      view.offsetY = screenCenterY - centerY * scale;
    }

    function worldToScreen(x, y) {
      return {
        x: x * view.scale + view.offsetX,
        y: y * view.scale + view.offsetY
      };
    }

    // Pan y zoom básico (táctil + mouse)
    let isPanning = false, lastPan = { x: 0, y: 0 };
    canvas.addEventListener('mousedown', (e) => {
      isPanning = true;
      lastPan = { x: e.clientX, y: e.clientY };
    });
    window.addEventListener('mouseup', () => isPanning = false);
    window.addEventListener('mousemove', (e) => {
      if (!isPanning) return;
      const dx = e.clientX - lastPan.x;
      const dy = e.clientY - lastPan.y;
      view.offsetX += dx * devicePixelRatio;
      view.offsetY += dy * devicePixelRatio;
      lastPan = { x: e.clientX, y: e.clientY };
      redrawAll();
    });
    canvas.addEventListener('wheel', (e) => {
      e.preventDefault();
      const factor = e.deltaY < 0 ?
