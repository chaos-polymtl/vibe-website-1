<script src="https://cdn.plot.ly/plotly-2.35.2.min.js"></script>

# Contrôle de niveau d'un réservoir (régulateur PI)

Cette application simule la **dynamique** d'un réservoir et son **contrôle par rétroaction**. Le réservoir est alimenté par le haut avec un débit $Q_{in}$ et se vide par le bas à travers une **vanne** dont l'ouverture est ajustée automatiquement par un **régulateur PI** (proportionnel + intégral) afin de maintenir le niveau à la valeur de consigne.

## Schéma du système

Avant d'écrire le bilan, on identifie les grandeurs en jeu sur un schéma du réservoir : le débit **entrant** $Q_{in}$ qui alimente le réservoir par le haut, le débit **sortant** $Q_{out}$ qui s'évacue par la vanne du bas, la **section** $A$ du réservoir et le **niveau** de liquide $h$.

<figure style="margin:1.2rem auto;max-width:340px;text-align:center;">
<svg viewBox="0 0 320 350" xmlns="http://www.w3.org/2000/svg" style="width:100%;height:auto;font-family:var(--md-text-font,sans-serif);">
  <defs>
    <linearGradient id="app2-schem-water" x1="0" y1="0" x2="0" y2="1">
      <stop offset="0%"  stop-color="#38bdf8"/>
      <stop offset="100%" stop-color="#0369a1"/>
    </linearGradient>
    <marker id="app2-schem-arrow" markerWidth="13" markerHeight="13" refX="9" refY="6"
            orient="auto" markerUnits="userSpaceOnUse">
      <path d="M0,0 L11,6 L0,12 Z" fill="#22c55e"/>
    </marker>
    <marker id="app2-schem-arrow-out" markerWidth="13" markerHeight="13" refX="9" refY="6"
            orient="auto" markerUnits="userSpaceOnUse">
      <path d="M0,0 L11,6 L0,12 Z" fill="#f97316"/>
    </marker>
  </defs>

  <!-- ── INLET ──────────────────────────────────────────── -->
  <!-- inlet pipe (grey stub entering the tank top) -->
  <rect x="156" y="60" width="18" height="32" fill="#94a3b8"/>
  <!-- Q_in flow arrow (points down into the tank) -->
  <line x1="165" y1="26" x2="165" y2="58" stroke="#22c55e" stroke-width="5"
        marker-end="url(#app2-schem-arrow)"/>
  <text x="184" y="48" font-size="18" fill="#16a34a" font-style="italic">Q<tspan font-size="12" dy="4">in</tspan></text>

  <!-- ── TANK ───────────────────────────────────────────── -->
  <!-- water fill -->
  <rect x="98" y="170" width="134" height="110" fill="url(#app2-schem-water)" opacity="0.9"/>
  <!-- surface line -->
  <line x1="95" y1="170" x2="235" y2="170" stroke="#bae6fd" stroke-width="1.5"/>
  <!-- tank outline (open top) -->
  <path d="M95,90 L95,280 L235,280 L235,90"
        fill="none" stroke="var(--md-default-fg-color)" stroke-width="3" stroke-linejoin="round"/>

  <!-- section A (dashed width near the top) -->
  <line x1="95" y1="112" x2="235" y2="112" stroke="var(--md-default-fg-color--light,#888)"
        stroke-width="1" stroke-dasharray="4 3"/>
  <text x="165" y="133" font-size="16" text-anchor="middle"
        fill="var(--md-default-fg-color)" font-style="italic">section A</text>

  <!-- level h: dimension line on the left -->
  <line x1="76" y1="280" x2="76" y2="170" stroke="var(--md-default-fg-color--light,#888)" stroke-width="1.5"/>
  <line x1="71" y1="280" x2="81" y2="280" stroke="var(--md-default-fg-color--light,#888)" stroke-width="1.5"/>
  <line x1="71" y1="170" x2="81" y2="170" stroke="var(--md-default-fg-color--light,#888)" stroke-width="1.5"/>
  <text x="66" y="230" font-size="18" text-anchor="end"
        fill="var(--md-default-fg-color)" font-style="italic">h</text>

  <!-- ── OUTLET ─────────────────────────────────────────── -->
  <!-- outlet pipe (grey stub leaving the tank bottom) -->
  <rect x="156" y="280" width="18" height="30" fill="#94a3b8"/>
  <!-- valve symbol (bow-tie) on the outlet pipe -->
  <path d="M153,288 L177,302 L177,288 L153,302 Z" fill="#f97316" stroke="#7c2d12" stroke-width="1.2"/>
  <!-- Q_out flow arrow (points down out of the valve) -->
  <line x1="165" y1="310" x2="165" y2="340" stroke="#f97316" stroke-width="5"
        marker-end="url(#app2-schem-arrow-out)"/>
  <text x="184" y="332" font-size="18" fill="#ea580c" font-style="italic">Q<tspan font-size="12" dy="4">out</tspan></text>
</svg>
<figcaption style="font-size:0.8rem;color:var(--md-default-fg-color--light);">
Réservoir alimenté par un débit <i>Q<sub>in</sub></i> et vidangé par une vanne (<i>Q<sub>out</sub></i>).
Le niveau <i>h</i> évolue selon le bilan de matière établi ci-dessous.
</figcaption>
</figure>

## Dynamique du procédé

On établit la dynamique du réservoir à partir d'un **bilan de matière** général. Sur le contenu du réservoir, le bilan s'écrit sous sa forme la plus générale :

$$\text{Accumulation} = \text{Entrée} - \text{Sortie}$$

c'est-à-dire que la masse accumulée dans le réservoir varie au rythme de la différence entre le débit massique entrant et le débit massique sortant :

$$\frac{dm}{dt} = \dot{m}_{in} - \dot{m}_{out}$$

La masse contenue dans le réservoir (de section $A$) s'exprime par $m = \rho\, V = \rho\, A\, h$, et les débits massiques sont reliés aux débits volumiques par $\dot{m}_{in} = \rho\, Q_{in}$ et $\dot{m}_{out} = \rho\, Q_{out}$ :

$$\frac{d(\rho\, A\, h)}{dt} = \rho\, Q_{in} - \rho\, Q_{out}$$

En supposant la **masse volumique $\rho$ constante** (liquide incompressible) ainsi qu'une section $A$ constante, on peut sortir $\rho$ et $A$ de la dérivée puis simplifier par $\rho$. Le bilan de matière se ramène alors à un **bilan de volume** sur le niveau $h$ :

$$A\,\frac{dh}{dt} = Q_{in} - Q_{out}$$

Le débit à travers la vanne de sortie suit la **loi de Torricelli** : il dépend de l'ouverture de la vanne $u$ (entre 0 = fermée et 1 = grande ouverte) et de la **hauteur de liquide** au-dessus de la vanne (charge hydrostatique) :

$$Q_{out} = C_v\, u\, \sqrt{h}, \qquad u \in [0,\,1]$$

où $C_v$ est le coefficient de la vanne. Cette relation est **non linéaire** : pour une même ouverture, le réservoir se vide plus vite lorsqu'il est plein. Laissé seul (boucle ouverte), le système atteint un **régime permanent** lorsque $Q_{out} = Q_{in}$ ; avant cela, il traverse un **régime transitoire** caractérisé par une constante de temps qui dépend du point de fonctionnement.

## Principe de la rétroaction

En **boucle fermée**, on mesure le niveau $h$, on le compare à la **consigne** $h_{sp}$, et on agit sur la vanne pour corriger l'**écart** :

$$e(t) = h(t) - h_{sp}$$

La régulation est ici **directe** : si le niveau est trop haut ($e > 0$), on **ouvre davantage** la vanne pour évacuer plus de liquide ; s'il est trop bas, on la referme.

## Régulateur PI

Le régulateur **proportionnel–intégral** calcule l'ouverture de la vanne à partir de l'écart :

$$u(t) = K_p\, e(t) + K_i \int_0^{t} e(\tau)\,d\tau$$

- L'**action proportionnelle** ($K_p$) réagit à l'écart instantané. Utilisée seule (régulateur P), elle laisse subsister un **écart statique** (offset) : un écart résiduel permanent est nécessaire pour maintenir l'ouverture de vanne au bon niveau.
- L'**action intégrale** ($K_i$) accumule l'écart dans le temps et **annule l'écart statique** : tant qu'il reste un écart, le terme intégral continue d'évoluer jusqu'à ramener exactement le niveau à la consigne.

L'ouverture est physiquement **bornée** : $u$ est saturée entre 0 et 1. Lorsque la vanne sature, on gèle l'accumulation de l'intégrale pour éviter l'**emballement intégral** (*anti-windup*), qui provoquerait sinon de grands dépassements.

!!! tip "À expérimenter"
    - Décochez **Action intégrale**. Comment se comporte le contrôleur en régime permanent?
    - Appliquez une **perturbation** sur le débit d'entrée pour visualiser le **rejet de perturbation**.
    - Augmentez $K_p$ à de très grandes valeurs, quel comportement adopte le sytème?
    - Que pouvez-vous dire sur la valeur de l'**erreur statique** en fonctin du gain proportionel du système?
    - Dans quelles circonstances est-ce qu'un dépassement serait problématique?

---

<style>
#app2-container {
  margin-top: 1rem;
}
/* Card-style panels */
.app2-panel {
  background: var(--md-code-bg-color);
  border: 1px solid var(--md-default-fg-color--lightest, rgba(0,0,0,0.1));
  border-left: 3px solid #6366f1;
  border-radius: 8px;
  padding: 0.9rem 1.1rem;
  margin-bottom: 0.9rem;
}
.app2-panel-title {
  font-size: 0.75rem;
  font-weight: 700;
  letter-spacing: 0.08em;
  text-transform: uppercase;
  color: var(--md-default-fg-color--light);
  margin-bottom: 0.7rem;
}
/* Inputs row */
.app2-inputs {
  display: flex;
  flex-wrap: wrap;
  gap: 1.2rem;
  align-items: flex-end;
}
.app2-inputs label {
  display: flex;
  flex-direction: column;
  font-size: 0.8rem;
  gap: 0.25rem;
  color: var(--md-default-fg-color--light);
}
.app2-inputs input[type="number"] {
  width: 96px;
  padding: 0.3rem 0.5rem;
  border: 1px solid var(--md-default-fg-color--lighter);
  border-radius: 5px;
  background: var(--md-default-bg-color);
  color: var(--md-default-fg-color);
  font-size: 0.88rem;
  transition: border-color 0.15s;
}
.app2-inputs input[type="number"]:focus {
  outline: none;
  border-color: var(--md-accent-fg-color, teal);
}
.app2-check {
  flex-direction: row !important;
  align-items: center;
  gap: 0.45rem !important;
  cursor: pointer;
  user-select: none;
}
.app2-check input { width: auto; }
/* Buttons */
.app2-btn {
  background: linear-gradient(135deg, #0ea5e9 0%, #6366f1 100%);
  color: #fff;
  border: none;
  padding: 0.55rem 1.4rem;
  border-radius: 6px;
  cursor: pointer;
  font-size: 0.9rem;
  font-weight: 600;
  letter-spacing: 0.02em;
  margin-right: 0.6rem;
  margin-bottom: 0.5rem;
  box-shadow: 0 2px 8px rgba(99,102,241,0.35);
  transition: transform 0.1s ease, box-shadow 0.15s ease, opacity 0.15s;
}
.app2-btn:hover {
  opacity: 0.92;
  transform: translateY(-2px);
  box-shadow: 0 5px 16px rgba(99,102,241,0.45);
}
.app2-btn:active {
  transform: translateY(0);
  box-shadow: 0 1px 4px rgba(99,102,241,0.3);
}
.app2-btn.app2-btn-alt {
  background: linear-gradient(135deg, #f59e0b 0%, #f97316 100%);
  box-shadow: 0 2px 8px rgba(249,115,22,0.35);
}
.app2-btn.app2-btn-ghost {
  background: transparent;
  color: var(--md-default-fg-color--light);
  border: 1px solid var(--md-default-fg-color--lighter);
  box-shadow: none;
}
/* Status line */
#app2-status {
  font-size: 0.82rem;
  font-family: var(--md-code-font, monospace);
  color: var(--md-default-fg-color--light);
  margin: 0.5rem 0 0.2rem;
  min-height: 1.3em;
}
/* Live badge */
@keyframes app2-pulse {
  0%, 100% { opacity: 1;   transform: scale(1);   }
  50%      { opacity: 0.3; transform: scale(0.7); }
}
#app2-live-row {
  display: flex;
  align-items: center;
  gap: 0.6rem;
  flex-wrap: wrap;
  margin-bottom: 0.2rem;
}
#app2-live-badge {
  display: inline-flex;
  align-items: center;
  gap: 0.35rem;
  font-size: 0.72rem;
  font-weight: 600;
  letter-spacing: 0.06em;
  text-transform: uppercase;
  color: #94a3b8;
}
#app2-live-badge.running { color: #22c55e; }
#app2-live-dot {
  width: 7px; height: 7px;
  border-radius: 50%;
  background: currentColor;
}
#app2-live-badge.running #app2-live-dot {
  animation: app2-pulse 1s ease-in-out infinite;
}
/* Visualization */
#app2-viz {
  display: flex;
  flex-wrap: wrap;
  gap: 1rem;
  margin-top: 0.6rem;
  align-items: stretch;
}
#app2-reservoir {
  flex: 0 0 260px;
  min-width: 240px;
  border-radius: 8px;
  background: var(--md-code-bg-color);
  box-shadow: 0 4px 24px rgba(0,0,0,0.08);
  padding: 0.5rem;
}
#app2-reservoir svg { width: 100%; height: auto; display: block; }
.app2-plot {
  flex: 1;
  min-width: 320px;
  min-height: 270px;
  border-radius: 8px;
  overflow: hidden;
  box-shadow: 0 4px 24px rgba(99,102,241,0.18), 0 2px 8px rgba(0,0,0,0.07);
}
</style>

<div id="app2-container">

  <div class="app2-panel">
    <div class="app2-panel-title">Procédé — réservoir &amp; vanne</div>
    <div class="app2-inputs">
      <label>Section A (m²)
        <input type="number" id="app2-area" value="1.0" step="0.1" min="0.1">
      </label>
      <label>Hauteur max H (m)
        <input type="number" id="app2-hmax" value="2.0" step="0.1" min="0.2">
      </label>
      <label>Coefficient vanne Cv
        <input type="number" id="app2-cv" value="0.5" step="0.1" min="0.05">
      </label>
      <label>Débit entrée Q<sub>in</sub> (m³/s)
        <input type="number" id="app2-qin" value="0.3" step="0.1" min="0">
      </label>
    </div>
  </div>

  <div class="app2-panel">
    <div class="app2-panel-title">Régulateur PI</div>
    <div class="app2-inputs">
      <label>Consigne h<sub>sp</sub> (m)
        <input type="number" id="app2-hsp" value="1.0" step="0.1" min="0">
      </label>
      <label>Gain proportionnel K<sub>p</sub>
        <input type="number" id="app2-kp" value="2.0" step="0.1">
      </label>
      <label>Gain intégral K<sub>i</sub>
        <input type="number" id="app2-ki" value="0.5" step="0.1">
      </label>
      <label>Vitesse (×)
        <input type="number" id="app2-speed" value="1.0" step="0.1" min="0.1" max="20">
      </label>
      <label class="app2-check">
        <input type="checkbox" id="app2-pi" checked>
        Action intégrale (PI)
      </label>
    </div>
  </div>

  <div id="app2-live-row">
    <button class="app2-btn" id="app2-toggle-btn" onclick="app2Toggle()">Démarrer</button>
    <button class="app2-btn app2-btn-alt" onclick="app2Disturb()">Appliquer une perturbation</button>
    <button class="app2-btn app2-btn-ghost" onclick="app2Reset()">Réinitialiser</button>
    <span id="app2-live-badge">
      <span id="app2-live-dot"></span>
      <span id="app2-live-label">En pause</span>
    </span>
  </div>

  <p id="app2-status"></p>

  <div id="app2-viz">
    <div id="app2-reservoir"></div>
    <div id="app2-plot-level" class="app2-plot"></div>
    <div id="app2-plot-control" class="app2-plot"></div>
  </div>

</div>

<script>
(function () {

  // ── Parameters ───────────────────────────────────────────────────────────────
  function getParams() {
    const n = (id, d) => {
      const v = parseFloat(document.getElementById(id).value);
      return isNaN(v) ? d : v;
    };
    return {
      A:     Math.max(0.05, n('app2-area', 1.0)),
      Hmax:  Math.max(0.2,  n('app2-hmax', 2.0)),
      Cv:    Math.max(0.01, n('app2-cv',   0.5)),
      Qin:   Math.max(0,    n('app2-qin',  0.3)),
      hsp:   Math.max(0,    n('app2-hsp',  1.0)),
      Kp:    n('app2-kp', 2.0),
      Ki:    n('app2-ki', 0.5),
      speed: Math.min(20, Math.max(0.1, n('app2-speed', 1.0))),
      pi:    document.getElementById('app2-pi').checked,
    };
  }

  // ── Simulation state ───────────────────────────────────────────────────────────
  const DT = 0.02;          // fixed internal time step (s)
  const WINDOW = 60;        // rolling plot window (s)
  let st;
  function initState() {
    st = { h: 0, integral: 0, u: 0, t: 0, Qout: 0, running: false,
           T: [], H: [], SP: [], U: [], QIN: [], QOUT: [] };
  }
  initState();

  const clamp = (x, lo, hi) => Math.min(hi, Math.max(lo, x));

  // ── One physical sub-step (controller + plant) ──────────────────────────────────
  function step(p) {
    const e = st.h - p.hsp;                       // direct-acting error
    const P = p.Kp * e;
    let I = 0;
    if (p.pi) {
      const uTrial = P + p.Ki * (st.integral + e * DT);
      // Anti-windup: only integrate when not pushing further into saturation
      if (!((uTrial > 1 && e > 0) || (uTrial < 0 && e < 0))) {
        st.integral += e * DT;
      }
      I = p.Ki * st.integral;
    } else {
      st.integral = 0;
    }
    st.u = clamp(P + I, 0, 1);

    st.Qout = p.Cv * st.u * Math.sqrt(Math.max(st.h, 0));
    st.h = clamp(st.h + (p.Qin - st.Qout) / p.A * DT, 0, p.Hmax);
    st.t += DT;
  }

  function record(p) {
    st.T.push(st.t); st.H.push(st.h); st.SP.push(p.hsp);
    st.U.push(st.u * 100); st.QIN.push(p.Qin); st.QOUT.push(st.Qout);
    while (st.T.length && st.t - st.T[0] > WINDOW) {
      st.T.shift(); st.H.shift(); st.SP.shift();
      st.U.shift(); st.QIN.shift(); st.QOUT.shift();
    }
  }

  // ── Reservoir SVG drawing ───────────────────────────────────────────────────────
  function drawReservoir(p) {
    const W = 240, Hpx = 300, pad = 30;
    const tankX = 70, tankW = 110, tankTop = pad, tankBot = Hpx - pad;
    const tankH = tankBot - tankTop;
    const frac = clamp(st.h / p.Hmax, 0, 1);
    const waterTop = tankBot - frac * tankH;
    const spY = tankBot - clamp(p.hsp / p.Hmax, 0, 1) * tankH;
    const inFlow = p.Qin > 1e-6;
    const valvePct = Math.round(st.u * 100);
    // valve colour: closed→open  amber→cyan
    const valveCol = `rgb(${Math.round(245 - st.u*231)}, ${Math.round(158 + st.u*7)}, ${Math.round(11 + st.u*222)})`;

    document.getElementById('app2-reservoir').innerHTML = `
    <svg viewBox="0 0 ${W} ${Hpx}" xmlns="http://www.w3.org/2000/svg">
      <defs>
        <linearGradient id="app2-water" x1="0" y1="0" x2="0" y2="1">
          <stop offset="0%"  stop-color="#38bdf8"/>
          <stop offset="100%" stop-color="#0369a1"/>
        </linearGradient>
      </defs>
      <!-- inlet pipe -->
      <rect x="${tankX+tankW/2-9}" y="0" width="18" height="${tankTop}" fill="#64748b"/>
      ${inFlow ? `<rect x="${tankX+tankW/2-4}" y="2" width="8" height="${tankTop-2}" fill="#38bdf8" opacity="0.85"/>` : ``}
      <!-- water -->
      <rect x="${tankX+3}" y="${waterTop}" width="${tankW-6}" height="${tankBot-waterTop}" fill="url(#app2-water)"/>
      <!-- setpoint line -->
      <line x1="${tankX-6}" y1="${spY}" x2="${tankX+tankW+6}" y2="${spY}"
            stroke="#ef4444" stroke-width="2" stroke-dasharray="6 4"/>
      <text x="${tankX+tankW+9}" y="${spY+4}" font-size="11" fill="#ef4444">h&#8203;sp</text>
      <!-- tank outline -->
      <rect x="${tankX}" y="${tankTop}" width="${tankW}" height="${tankH}"
            fill="none" stroke="var(--md-default-fg-color--light)" stroke-width="2.5"/>
      <!-- outlet pipe + valve -->
      <rect x="${tankX+tankW/2-7}" y="${tankBot}" width="14" height="${pad}" fill="#64748b"/>
      <circle cx="${tankX+tankW/2}" cy="${tankBot+pad/2}" r="9" fill="${valveCol}" stroke="#1e293b" stroke-width="1.5"/>
      <text x="${tankX+tankW/2+16}" y="${tankBot+pad/2+4}" font-size="11" fill="var(--md-default-fg-color--light)">${valvePct}%</text>
      <!-- level readout -->
      <text x="${tankX+tankW/2}" y="${tankTop-10}" font-size="13" text-anchor="middle"
            font-weight="600" fill="var(--md-default-fg-color)">h = ${st.h.toFixed(3)} m</text>
    </svg>`;
  }

  // ── Plot layout helpers (transparent / theme-neutral) ───────────────────────────
  const AXIS = {
    zeroline: false, color: '#999',
    gridcolor: 'rgba(150,150,150,0.15)', linecolor: 'rgba(150,150,150,0.3)',
  };
  function plotLayout(extra) {
    return Object.assign({
      paper_bgcolor: 'rgba(0,0,0,0)', plot_bgcolor: 'rgba(0,0,0,0)',
      font: { color: '#888', size: 11 },
      margin: { t: 40, l: 52, r: 16, b: 42 },
      hoverlabel: { bgcolor: '#1e293b', font: { color: '#f1f5f9', size: 12 } },
      legend: { orientation: 'h', y: 1.12, x: 0, font: { size: 10 } },
    }, extra);
  }

  let plotRev = 0;
  function drawPlots(p) {
    const xr = [Math.max(0, st.t - WINDOW), Math.max(WINDOW, st.t)];
    // datarevision forces Plotly.react to re-read the in-place-mutated data arrays
    // (without it, react diffs by array reference and skips the update)
    const rev = ++plotRev;
    Plotly.react('app2-plot-level', [
      { x: st.T, y: st.SP, name: 'Consigne', mode: 'lines',
        line: { color: '#ef4444', width: 2, dash: 'dash' }, hoverinfo: 'skip' },
      { x: st.T, y: st.H, name: 'Niveau h', mode: 'lines',
        line: { color: '#0ea5e9', width: 2.5 },
        hovertemplate: 't=%{x:.1f}s<br>h=%{y:.3f} m<extra></extra>' },
    ], plotLayout({
      datarevision: rev,
      title: { text: 'Niveau du réservoir', font: { size: 13 } },
      xaxis: Object.assign({}, AXIS, { title: { text: 't (s)' }, range: xr }),
      yaxis: Object.assign({}, AXIS, { title: { text: 'h (m)' }, range: [0, p.Hmax] }),
    }), { responsive: true });

    Plotly.react('app2-plot-control', [
      { x: st.T, y: st.QIN, name: 'Q entrée', mode: 'lines',
        line: { color: '#22c55e', width: 2 }, yaxis: 'y2',
        hovertemplate: 't=%{x:.1f}s<br>Qin=%{y:.3f}<extra></extra>' },
      { x: st.T, y: st.QOUT, name: 'Q sortie', mode: 'lines',
        line: { color: '#f97316', width: 2 }, yaxis: 'y2',
        hovertemplate: 't=%{x:.1f}s<br>Qout=%{y:.3f}<extra></extra>' },
      { x: st.T, y: st.U, name: 'Ouverture vanne', mode: 'lines',
        line: { color: '#6366f1', width: 2.5 },
        hovertemplate: 't=%{x:.1f}s<br>u=%{y:.1f}%<extra></extra>' },
    ], plotLayout({
      datarevision: rev,
      title: { text: 'Commande &amp; débits', font: { size: 13 } },
      xaxis: Object.assign({}, AXIS, { title: { text: 't (s)' }, range: xr }),
      yaxis: Object.assign({}, AXIS, { title: { text: 'vanne (%)' }, range: [0, 100] }),
      yaxis2: Object.assign({}, AXIS, { title: { text: 'Q (m³/s)' },
        overlaying: 'y', side: 'right', rangemode: 'tozero', showgrid: false }),
      margin: { t: 40, l: 52, r: 52, b: 42 },
    }), { responsive: true });
  }

  // ── Status line ────────────────────────────────────────────────────────────────
  function updateStatus(p) {
    const e = st.h - p.hsp;
    document.getElementById('app2-status').textContent =
      `t = ${st.t.toFixed(1)} s   |   h = ${st.h.toFixed(3)} m   |   écart e = ${e.toFixed(3)} m   ` +
      `|   vanne u = ${(st.u*100).toFixed(1)} %   |   Qin = ${p.Qin.toFixed(3)}   Qout = ${st.Qout.toFixed(3)} m³/s`;
  }

  function setBadge() {
    const b = document.getElementById('app2-live-badge');
    const l = document.getElementById('app2-live-label');
    const btn = document.getElementById('app2-toggle-btn');
    b.className = st.running ? 'running' : '';
    l.textContent = st.running ? 'En cours' : 'En pause';
    btn.textContent = st.running ? 'Pause' : 'Démarrer';
  }

  function renderAll(p) { drawReservoir(p); drawPlots(p); updateStatus(p); }

  // ── Animation loop (setInterval: keeps a steady cadence and survives a
  //    transient exception in one tick, unlike a self-rescheduling rAF) ────────────
  const TICK_MS = 40;          // wall-clock render period (≈25 fps)
  let timer = null, frame = 0;
  function tick() {
    const p = getParams();
    const simSeconds = (TICK_MS / 1000) * p.speed;
    const n = Math.max(1, Math.round(simSeconds / DT));
    for (let k = 0; k < n; k++) step(p);
    record(p);
    drawReservoir(p);
    updateStatus(p);
    if (frame++ % 2 === 0) drawPlots(p);   // throttle plot redraws
  }
  function startTimer() { if (!timer) timer = setInterval(tick, TICK_MS); }
  function stopTimer()  { if (timer) { clearInterval(timer); timer = null; } }

  // ── Public controls ──────────────────────────────────────────────────────────────
  window.app2Toggle = function () {
    st.running = !st.running;
    setBadge();
    if (st.running) startTimer();
    else { stopTimer(); drawPlots(getParams()); }
  };

  window.app2Reset = function () {
    stopTimer();
    initState();
    setBadge();
    renderAll(getParams());
  };

  window.app2Disturb = function () {
    const el = document.getElementById('app2-qin');
    const cur = parseFloat(el.value) || 0;
    el.value = (cur > 0 ? cur * 1.5 : 0.3).toFixed(3);
    if (!st.running) renderAll(getParams());
  };

  // ── Live-editable parameters (no reset required) ─────────────────────────────────
  setTimeout(function () {
    ['app2-area','app2-hmax','app2-cv','app2-qin','app2-hsp',
     'app2-kp','app2-ki','app2-speed','app2-pi'].forEach(function (id) {
      const el = document.getElementById(id);
      if (el) el.addEventListener('input', function () { if (!st.running) renderAll(getParams()); });
    });
    setBadge();
    renderAll(getParams());
  }, 80);

})();
</script>
