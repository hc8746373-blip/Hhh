<!doctype html>
<html lang="en">
<head>
  <!--
  index.html - High-end Mechanics Calculator (single-file)
  -----------------------------------------------------
  Developer notes:
  - Single-file app. Open in browser (double-click index.html) — no server required.
  - CDN libraries (lazy-loaded when needed): math.js, Plotly, KaTeX, html2pdf.
  - To tweak accent color: change --accent variable in :root.
  - For PDF export, html2pdf is recommended (lazy-loaded).
  - Two example presets are preloaded: Free fall and Projectile example.
  - Accessibility: inputs have labels, aria attributes, keyboard shortcuts:
      Enter => compute, Ctrl+L => clear, / => focus preset search.
  - Performance: heavy libraries loaded on demand.
  - Recommended deployment: static hosting (GitHub Pages, Netlify).
  -----------------------------------------------------
  -->
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>HARSH 02 — Mechanics Calculator</title>

  <!-- Google Fonts -->
  <link href="https://fonts.googleapis.com/css2?family=Orbitron:wght@600;700;900&family=Inter:wght@300;400;600&family=Roboto+Mono&display=swap" rel="stylesheet">

  <!-- basic css -->
  <style>
    :root{
      --bg:#071126;
      --panel: rgba(255,255,255,0.04);
      --glass: rgba(255,255,255,0.03);
      --accent: #00f0ff; /* tweak this */
      --accent2: #b27bff;
      --muted: rgba(255,255,255,0.6);
      --glass-border: rgba(255,255,255,0.06);
      --neon: drop-shadow(0 0 8px rgba(0,240,255,0.16)) drop-shadow(0 0 16px rgba(178,123,255,0.06));
      --max-width: 1200px;
    }

    /* Basic reset */
    *{box-sizing:border-box}
    html,body{height:100%}
    body{
      margin:0;
      font-family: 'Inter', system-ui, -apple-system, "Segoe UI", Roboto, "Helvetica Neue", Arial;
      background: radial-gradient(ellipse at 10% 10%, rgba(40,10,60,0.18), transparent 10%),
                  linear-gradient(180deg, rgba(5,8,18,1) 0%, rgba(6,10,22,1) 40%, rgba(1,6,20,1) 100%);
      color: #e6f0ff;
      -webkit-font-smoothing:antialiased;
      -moz-osx-font-smoothing:grayscale;
      overflow-y: auto;
    }

    /* Starfield + faint grid + animated gradient for sci-fi background */
    body::before{
      content:'';
      position:fixed; inset:0; z-index:-3;
      background-image:
        radial-gradient(circle at 10% 10%, rgba(255,255,255,0.02), transparent 1px),
        radial-gradient(circle at 40% 30%, rgba(255,255,255,0.01), transparent 1px),
        radial-gradient(circle at 80% 70%, rgba(255,255,255,0.02), transparent 1px);
      background-size: 1200px 1200px, 900px 900px, 1000px 1000px;
      animation: drift 30s linear infinite;
      opacity:0.9;
      pointer-events:none;
    }
    @keyframes drift{from{transform:translateY(0)}to{transform:translateY(-80px)}}

    body::after{
      content:'';
      position:fixed; inset:0; z-index:-2;
      background: linear-gradient(120deg, rgba(0,240,255,0.03), rgba(178,123,255,0.02));
      mix-blend-mode:screen;
      opacity:0.7;
      pointer-events:none;
    }

    /* Container */
    .app{
      max-width: var(--max-width);
      margin: 28px auto;
      padding: 20px;
      display:grid;
      grid-template-columns: 420px 1fr;
      gap: 22px;
    }
    @media (max-width:880px){
      .app{grid-template-columns:1fr; padding: 12px}
    }

    /* Header */
    .header{
      grid-column: 1/-1;
      display:flex;
      align-items:center;
      gap:16px;
    }
    .brand{
      display:flex;
      align-items:center;
      gap:12px;
    }
    .brand .badge{
      font-family: 'Orbitron', sans-serif;
      font-size:46px;
      letter-spacing:6px;
      padding:18px 22px;
      border-radius:14px;
      background: linear-gradient(180deg, rgba(255,255,255,0.02), rgba(255,255,255,0.01));
      border: 1px solid rgba(255,255,255,0.06);
      box-shadow: 0 6px 20px rgba(0,0,0,0.6), inset 0 0 30px rgba(0,240,255,0.02);
      color:var(--accent);
      text-shadow: 0 0 10px rgba(0,240,255,0.12), 0 2px 6px rgba(0,0,0,0.7);
      filter: var(--neon);
      display:inline-block;
    }
    .brand .subtitle{
      font-size:14px;
      color:var(--muted);
      margin-top:8px;
    }

    /* top controls row */
    .top-controls{
      margin-left:auto;
      display:flex;
      gap:10px;
      align-items:center;
    }
    .glass{
      background:var(--glass);
      border-radius:12px;
      border: 1px solid var(--glass-border);
      padding:12px;
      backdrop-filter: blur(6px) saturate(120%);
    }

    /* Panels */
    .panel{
      background: linear-gradient(180deg, rgba(255,255,255,0.02), rgba(255,255,255,0.01));
      border-radius:14px;
      border: 1px solid rgba(255,255,255,0.04);
      padding:14px;
      box-shadow: 0 8px 30px rgba(2,4,12,0.6);
    }

    /* Left column controls */
    .controls{
      display:flex;
      flex-direction:column;
      gap:12px;
      position:relative;
    }

    .module{
      padding:12px;
      border-radius:12px;
      background: linear-gradient(180deg, rgba(255,255,255,0.01), rgba(255,255,255,0.005));
      border:1px solid rgba(255,255,255,0.03);
    }

    .module h3{
      margin:0 0 8px 0;
      font-size:16px;
      font-weight:600;
      color:var(--accent);
      font-family: 'Orbitron', sans-serif;
      letter-spacing:1px;
    }

    label{display:block; font-size:13px; color:var(--muted); margin-bottom:6px}
    .row{display:flex; gap:8px; align-items:center}
    input[type="text"], select, input[type="number"]{
      width:100%;
      padding:8px 10px;
      border-radius:8px;
      border:1px solid rgba(255,255,255,0.06);
      background: rgba(0,0,0,0.35);
      color:#e9f6ff;
      font-size:14px;
      outline:none;
      box-shadow: inset 0 -1px 0 rgba(255,255,255,0.02);
    }
    input:focus, select:focus{box-shadow: 0 0 0 4px rgba(0,240,255,0.06); border-color:var(--accent)}

    .unit-field{width:110px}
    .small{width:80px}
    .btn{
      display:inline-flex; align-items:center; gap:8px;
      padding:8px 12px;
      border-radius:10px;
      background: linear-gradient(180deg, rgba(255,255,255,0.02), rgba(255,255,255,0.01));
      border:1px solid rgba(255,255,255,0.06);
      cursor:pointer;
      color: white;
      font-weight:600;
      transition: transform .08s ease, box-shadow .12s ease;
      user-select:none;
    }
    .btn:hover{transform:translateY(-2px); box-shadow:0 12px 30px rgba(0,0,0,0.6)}
    .btn:active{transform:translateY(0)}
    .btn-primary{
      background: linear-gradient(90deg, var(--accent), var(--accent2));
      box-shadow: 0 6px 30px rgba(0,240,255,0.08);
      color: #02101b;
    }

    .muted{color:var(--muted); font-size:13px}

    /* Right column: results */
    .results{
      display:flex;
      flex-direction:column;
      gap:12px;
      min-height:420px;
    }
    .result-top{
      display:flex;
      gap:12px;
      align-items:flex-start;
      justify-content:space-between;
    }
    .numeric{
      font-family: 'Roboto Mono', monospace;
      font-size:22px;
      background: linear-gradient(180deg, rgba(255,255,255,0.01), rgba(255,255,255,0.00));
      padding:12px;
      border-radius:10px;
      border:1px solid rgba(255,255,255,0.04);
    }
    .steps{
      margin-top:6px;
      padding:10px;
      background: linear-gradient(180deg, rgba(0,0,0,0.3), rgba(255,255,255,0.01));
      border-radius:10px;
      max-height:360px;
      overflow:auto;
    }
    .formula{
      font-family: 'Roboto Mono', monospace;
      font-size:14px;
      padding:10px;
      border-radius:8px;
      border:1px dashed rgba(255,255,255,0.04);
      background: linear-gradient(180deg, rgba(0,0,0,0.22), rgba(255,255,255,0.01));
    }
    .plot{
      height:300px;
      border-radius:10px;
      overflow:hidden;
    }

    .collapsible{
      margin-top:6px;
      font-size:13px;
      color:var(--muted);
      cursor:pointer;
    }
    .panel-collapsed{display:none}

    .warning{
      color:#ffd1a1;
      background: linear-gradient(180deg, rgba(255,255,255,0.01), rgba(255,255,255,0.00));
      border-left:4px solid #ff9f43;
      padding:8px;
      border-radius:6px;
      font-size:13px;
    }

    .success{color:#b8ffd6}

    /* Presets & search */
    .presets{
      display:flex; gap:8px; flex-wrap:wrap;
    }
    .preset-card{
      padding:8px 10px; border-radius:8px; background:linear-gradient(180deg, rgba(255,255,255,0.01), rgba(255,255,255,0.005));
      border:1px solid rgba(255,255,255,0.03);
      cursor:pointer; font-size:13px;
    }

    footer{grid-column:1/-1; text-align:center; color:var(--muted); margin-top:18px; font-size:13px}
    .kbd{font-family:'Roboto Mono'; background:rgba(255,255,255,0.03); padding:6px 8px;border-radius:6px; font-size:12px}

    /* minimal/print mode toggle */
    .minimal .brand .badge{filter:none; color:#cfd8ff}
    .minimal body::after{display:none}
    .minimal .panel{border:none; box-shadow:none; background:#071126}
  </style>
</head>
<body>
  <div class="app" id="app">
    <div class="header">
      <div class="brand">
        <div class="badge">HARSH 02</div>
        <div>
          <div style="font-weight:600; font-size:14px; color:var(--muted)">Mechanics Calculator — JEE/Advanced level helper</div>
          <div class="subtitle muted">Sci-fi theme • Step-by-step Hinglish explanations • Units-aware</div>
        </div>
      </div>

      <div class="top-controls" role="region" aria-label="top controls">
        <div class="glass panel" style="display:flex; gap:8px; align-items:center;">
          <label for="themeToggle" class="muted" style="margin:0 8px 0 0">Theme</label>
          <select id="themeToggle" aria-label="Theme toggle">
            <option value="sci">Sci-Fi (default)</option>
            <option value="minimal">Minimal Dark (print)</option>
          </select>
          <button class="btn" id="explainShortBtn" title="Short explanation (3 lines)">Explain shorter</button>
          <button class="btn btn-primary" id="computeBtn" title="Compute (Enter)">Compute</button>
        </div>
      </div>
    </div>

    <!-- LEFT: Controls / Modules -->
    <div class="controls" aria-label="Calculator controls">

      <!-- Search / Preset -->
      <div class="panel module">
        <div style="display:flex; justify-content:space-between; align-items:center;">
          <h3>Presets & Quick Problems</h3>
          <div class="muted">Press <span class="kbd">/</span> to focus</div>
        </div>

        <div style="margin-top:8px; display:flex; gap:8px;">
          <input id="presetSearch" placeholder="Search presets (e.g. free fall)" aria-label="Preset search" />
          <button class="btn" id="clearBtn" title="Clear inputs (Ctrl+L)">Clear</button>
        </div>

        <div style="margin-top:10px" class="presets" id="presetList" aria-live="polite">
          <!-- preset cards inserted by JS -->
        </div>
      </div>

      <!-- Module selector -->
      <div class="panel module">
        <h3>Module</h3>
        <div style="display:flex; gap:8px; margin-bottom:8px;">
          <select id="moduleSelect" aria-label="Select module">
            <option value="kinematics1d">1D Kinematics (constant a)</option>
            <option value="projectile">Projectile motion</option>
            <option value="newton2">Newton's 2nd law</option>
            <option value="workenergy">Work & Energy</option>
            <option value="rotational">Rotational (I & torque)</option>
          </select>
          <select id="solveFor" aria-label="Solve for variable"></select>
        </div>

        <div class="muted">Pick a module; inputs below will change. Use keyboard Enter to compute.</div>
      </div>

      <!-- Inputs dynamic area -->
      <div id="inputsArea" class="module panel" aria-live="polite">
        <!-- dynamic module inputs inserted here -->
      </div>

      <!-- Assumptions & Units -->
      <div class="panel module">
        <h3>Assumptions & Units</h3>
        <div style="display:flex; gap:8px; align-items:center">
          <label for="gValue">g</label>
          <input id="gValue" value="9.81" aria-label="Gravitational acceleration value" />
          <div class="muted">m/s²</div>
        </div>
        <div style="margin-top:8px">
          <button class="btn" id="toggleUnitsPanel">Show units panel</button>
        </div>
        <div id="unitsPanel" class="panel-collapsed" style="margin-top:10px">
          <div class="muted">Unit conversions and assumptions shown here. All internal calc => SI.</div>
          <div style="margin-top:8px">
            <label>Preferred length unit</label>
            <select id="prefLength">
              <option value="m">m</option>
              <option value="cm">cm</option>
              <option value="mm">mm</option>
            </select>
          </div>
        </div>
      </div>

      <!-- Export -->
      <div class="panel module" style="display:flex; gap:8px; flex-direction:column;">
        <h3>Export / Save</h3>
        <div style="display:flex; gap:8px;">
          <button class="btn" id="copyTextBtn">Copy solution</button>
          <button class="btn" id="exportPdfBtn">Export PDF</button>
        </div>
        <div style="margin-top:8px; display:flex; gap:8px;">
          <button class="btn" id="exportJsonBtn">Export inputs (JSON)</button>
          <button class="btn" id="importJsonBtn">Import JSON</button>
        </div>
      </div>

      <div class="muted" style="font-size:13px; margin-top:6px">Shortcuts: Enter=Compute • Ctrl+L=Clear • /=Preset search • Ctrl+S=Export PDF</div>
    </div>

    <!-- RIGHT: Results -->
    <div class="results" aria-live="polite">
      <div class="panel result-top">
        <div>
          <div class="numeric" id="numericResult" role="status" aria-live="polite">Result: —</div>
          <div class="muted" style="margin-top:6px">Units and significant digits shown. Exact arithmetic retained until final rounding.</div>
        </div>

        <div style="min-width:220px">
          <div style="display:flex; gap:8px; justify-content:flex-end;">
            <div class="btn" id="symbolicToggle">Show symbolic</div>
            <div class="btn" id="plotToggle">Plot</div>
          </div>
          <div style="margin-top:8px; font-size:13px" class="muted">Symbolic output uses KaTeX (lazy loaded).</div>
        </div>
      </div>

      <div class="panel">
        <h3 style="margin-top:0">Step-by-step solution</h3>
        <div class="steps" id="stepsArea">
          <!-- Steps go here -->
        </div>
        <div style="margin-top:8px">
          <button class="btn" id="copyShortBtn">Copy short (3-line)</button>
          <button class="btn" id="toggleAssumptions">Assumptions & Units</button>
        </div>
      </div>

      <div class="panel">
        <h3 style="margin-top:0">Formula & Derivation</h3>
        <div class="formula" id="formulaArea">Select a module to see formulas and short derivations.</div>
      </div>

      <div class="panel">
        <h3 style="margin-top:0">Interactive Plot</h3>
        <div id="plotArea" class="plot" aria-label="Interactive plot area">No plot yet. Click Plot after computing.</div>
      </div>

    </div>

    <footer>
      No analytics • Uses math.js, KaTeX, Plotly via CDN (lazy). Built for students — explanations in Hinglish + English.
    </footer>
  </div>

  <!-- Lightweight base libs -->
  <script src="https://cdnjs.cloudflare.com/ajax/libs/mathjs/11.8.0/math.min.js" integrity="" crossorigin="anonymous"></script>

  <script>
  // App JS (plain ES6). Well-commented and modular.
  // Main responsibilities:
  //  - render inputs for each module
  //  - parse inputs with math.js (units)
  //  - compute numeric and symbolic results
  //  - render step-by-step Hinglish explanations in 6 steps
  //  - lazy-load KaTeX and Plotly when user requests symbolic/plot
  //  - provide presets and shortcuts

  (function(){
    // Short aliases
    const el = sel => document.querySelector(sel);
    const elAll = sel => Array.from(document.querySelectorAll(sel));

    // Globals & state
    const state = {
      module: 'kinematics1d',
      solveFor: null,
      lastSolution: null,
      kaTeXloaded: false,
      plotlyLoaded: false,
      presets: [],
    };

    // Utility: safe parse of unitful string via math.js
    function parseQuantity(str, defaultUnit=null){
      try{
        str = String(str).trim();
        if(str === '' || str == null) return null;
        // if plain number:
        if(/^[+-]?(\d+(\.\d*)?|\.\d+)(e[+-]?\d+)?$/i.test(str)){
          return {value: Number(str), unit: defaultUnit || '1'};
        }
        // try parse as math.unit or expression
        const u = math.unit(str);
        return {value: u.toNumber(u.units[0] ? u.units[0].unit.name : '1'), unit: u.formatUnits ? u.formatUnits() : (u.units ? u.toString().replace(/^[\d.\-+eE]+\s*/, '') : '') };
      }catch(e){
        // fallback: try to split number + token
        const m = String(str).match(/^([+-]?\d+(\.\d+)?(?:e[+-]?\d+)?)(.*)$/i);
        if(m){
          const num = Number(m[1]);
          const u = m[3].trim() || defaultUnit || '1';
          return {value: num, unit: u};
        }
        throw new Error('Could not parse quantity: '+str);
      }
    }

    // Convert a mathjs Unit to SI numeric (value in SI base)
    function toSIQuantity(q){
      // q can be {value, unit} or math.Unit
      try{
        if(q == null) return null;
        if(q && q.isUnit){ // math.Unit
          // get SI value for the first base unit type by converting to SI units with toSI?
          // We'll convert to SI base units by using toNumber with base dimension:
          // mathjs unit.toSI is not guaranteed; instead, convert to SI by .toNumber('m'), etc.
          // We'll produce object {value:number, unit:unitString}
          const str = q.toString(); // e.g. '5 kg'
          // return q.toSI()? Not always available. Use q.toNumber(q.units[0].unit.name) for single unit.
          if(q.units.length === 1){
            const base = q.units[0].unit.name;
            const n = q.toNumber(base);
            return {value: n, unit: base};
          } else {
            // multi-unit (e.g., N = kg m / s^2). Convert to SI by converting to SI prefixless string
            return {value: q.toNumber(q.formatUnits ? q.formatUnits() : ''), unit: q.formatUnits ? q.formatUnits() : q.toString().replace(/^[\d.\-+eE]+\s*/, '')};
          }
        } else if(q && typeof q === 'object' && 'value' in q && 'unit' in q){
          // Create math.unit and convert to SI by calling .toSI — math.js doesn't have toSI, so we will attempt common conversions:
          try{
            const munit = math.unit(q.value, q.unit);
            // If unit is dimensionless:
            if(munit.units.length===0) return {value:q.value, unit:'1'};
            // Convert each base unit to its SI representation. For simplicity, we'll convert to the simplest unit of same dimension:
            // We'll return munit.toNumber(munit.formatUnits()) if possible
            // Try to convert to SI by expressing in base units combined string:
            // Use munit.toSI? If fails fallback to toNumber of first unit
            if(typeof munit.toNumber === 'function'){
              // Attempt to produce number in SI for common dims: m, kg, s, rad, A, K, mol, cd
              // Try converting to SI by converting to each base ('m','kg','s') where applicable:
              const candidateBases = ['m','kg','s','A','K','mol','cd','rad'];
              for(const b of candidateBases){
                try{
                  const n = munit.toNumber(b);
                  // if no error, return
                  if(typeof n === 'number' && !isNaN(n)){
                    return {value:n, unit:b};
                  }
                }catch(_){}
              }
              // fallback: return numeric value and original unit string
 