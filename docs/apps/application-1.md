<script src="https://cdn.plot.ly/plotly-2.35.2.min.js"></script>

# Solveur de chaleur 2D

Cette application résout l'**équation de Poisson en 2D** sur un domaine rectangulaire :

$$\nabla^2 T = \frac{\partial^2 T}{\partial x^2} + \frac{\partial^2 T}{\partial y^2} = -S$$

La méthode des **différences finies** est utilisée sur une grille uniforme avec des conditions aux limites de Dirichlet dont la valeur est configurable sur chacun des quatre bords.

---

<style>
#app1-container {
  margin-top: 1rem;
}
/* Card-style panels */
.app1-panel {
  background: var(--md-code-bg-color);
  border: 1px solid var(--md-default-fg-color--lightest, rgba(0,0,0,0.1));
  border-radius: 8px;
  padding: 0.9rem 1.1rem;
  margin-bottom: 0.9rem;
}
.app1-panel-title {
  font-size: 0.75rem;
  font-weight: 700;
  letter-spacing: 0.08em;
  text-transform: uppercase;
  color: var(--md-default-fg-color--light);
  margin-bottom: 0.7rem;
}
/* Inputs row */
#app1-inputs {
  display: flex;
  flex-wrap: wrap;
  gap: 1.2rem;
  align-items: flex-end;
}
#app1-inputs label, #app1-bc-grid label {
  display: flex;
  flex-direction: column;
  font-size: 0.8rem;
  gap: 0.25rem;
  color: var(--md-default-fg-color--light);
}
#app1-inputs input[type="number"],
#app1-bc-grid input[type="number"] {
  padding: 0.3rem 0.5rem;
  border: 1px solid var(--md-default-fg-color--lighter);
  border-radius: 5px;
  background: var(--md-default-bg-color);
  color: var(--md-default-fg-color);
  font-size: 0.88rem;
  transition: border-color 0.15s;
}
#app1-inputs input[type="number"] { width: 88px; }
#app1-bc-grid input[type="number"] { width: 72px; text-align: center; }
#app1-inputs input[type="number"]:focus,
#app1-bc-grid input[type="number"]:focus {
  outline: none;
  border-color: var(--md-accent-fg-color, teal);
}
/* Buttons */
.app1-btn {
  background: linear-gradient(135deg, #0ea5e9 0%, #6366f1 100%);
  color: #fff;
  border: none;
  padding: 0.55rem 1.6rem;
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
.app1-btn:hover {
  opacity: 0.92;
  transform: translateY(-2px);
  box-shadow: 0 5px 16px rgba(99,102,241,0.45);
}
.app1-btn:active {
  transform: translateY(0);
  box-shadow: 0 1px 4px rgba(99,102,241,0.3);
}
/* Status line */
#app1-status {
  font-size: 0.82rem;
  font-family: var(--md-code-font, monospace);
  color: var(--md-default-fg-color--light);
  margin: 0.5rem 0 0.2rem;
  min-height: 1.3em;
}
/* Plots */
#app1-plots {
  display: flex;
  flex-wrap: wrap;
  gap: 1rem;
  margin-top: 0.6rem;
}
#app1-mesh-plot,
#app1-solution-plot {
  min-width: 340px;
  flex: 1;
  min-height: 390px;
  border-radius: 8px;
  overflow: hidden;
}
/* BC layout */
#app1-bc-grid {
  display: grid;
  grid-template-columns: auto auto auto;
  gap: 0.35rem 0.6rem;
  align-items: center;
  justify-items: center;
  width: fit-content;
}
#app1-bc-grid label { align-items: center; }
#app1-bc-domain {
  width: 78px;
  height: 50px;
  border: 2px dashed #f97316;
  border-radius: 5px;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 0.72rem;
  font-weight: 600;
  letter-spacing: 0.05em;
  color: #f97316;
  background: rgba(249,115,22,0.06);
}
/* Panel accent left border */
.app1-panel {
  border-left: 3px solid #6366f1;
}
/* Plot card shadows */
#app1-mesh-plot {
  box-shadow: 0 4px 24px rgba(0,0,0,0.08);
}
#app1-solution-plot {
  box-shadow: 0 4px 24px rgba(99,102,241,0.18), 0 2px 8px rgba(0,0,0,0.07);
}
/* Live auto-solve indicator */
@keyframes app1-pulse {
  0%, 100% { opacity: 1;   transform: scale(1);    }
  50%       { opacity: 0.3; transform: scale(0.7);  }
}
#app1-live-row {
  display: flex;
  align-items: center;
  gap: 0.6rem;
  flex-wrap: wrap;
  margin-bottom: 0.2rem;
}
#app1-live-badge {
  display: inline-flex;
  align-items: center;
  gap: 0.35rem;
  font-size: 0.72rem;
  font-weight: 600;
  letter-spacing: 0.06em;
  text-transform: uppercase;
  color: #22c55e;
  opacity: 0;
  transition: opacity 0.3s;
}
#app1-live-badge.active { opacity: 1; }
#app1-live-badge.pending { color: #f59e0b; }
#app1-live-badge.solving { color: #3b82f6; }
#app1-live-dot {
  width: 7px; height: 7px;
  border-radius: 50%;
  background: currentColor;
  animation: app1-pulse 1s ease-in-out infinite;
}
#app1-live-badge:not(.pending):not(.solving) #app1-live-dot {
  animation-duration: 2.5s;
}
</style>

<div id="app1-container">

  <div class="app1-panel">
    <div class="app1-panel-title">Paramètres du domaine</div>
    <div id="app1-inputs">
      <label>Nœuds en x (Nx)
        <input type="number" id="app1-nx" value="20" min="3" max="100">
      </label>
      <label>Nœuds en y (Ny)
        <input type="number" id="app1-ny" value="20" min="3" max="100">
      </label>
      <label>Longueur Lx
        <input type="number" id="app1-lx" value="1.0" step="0.1" min="0.1">
      </label>
      <label>Longueur Ly
        <input type="number" id="app1-ly" value="1.0" step="0.1" min="0.1">
      </label>
      <label>Terme source S
        <input type="number" id="app1-s" value="1.0" step="0.1">
      </label>
    </div>
  </div>

  <div class="app1-panel">
    <div class="app1-panel-title">Conditions aux limites (Dirichlet)</div>
    <div id="app1-bc-grid">
      <div></div>
      <label>Haut
        <input type="number" id="app1-t-top" value="0" step="1">
      </label>
      <div></div>
      <label>Gauche
        <input type="number" id="app1-t-left" value="0" step="1">
      </label>
      <div id="app1-bc-domain">Domaine</div>
      <label>Droite
        <input type="number" id="app1-t-right" value="0" step="1">
      </label>
      <div></div>
      <label>Bas
        <input type="number" id="app1-t-bottom" value="0" step="1">
      </label>
      <div></div>
    </div>
  </div>

  <div id="app1-live-row">
    <button class="app1-btn" onclick="app1ShowMesh()">Afficher le maillage</button>
    <button class="app1-btn" onclick="app1Solve()">Résoudre</button>
    <span id="app1-live-badge">
      <span id="app1-live-dot"></span>
      <span id="app1-live-label">En direct</span>
    </span>
  </div>

  <p id="app1-status"></p>

  <div id="app1-plots">
    <div id="app1-mesh-plot"></div>
    <div id="app1-solution-plot"></div>
  </div>

</div>

<script>
(function () {

  // ── Inputs ──────────────────────────────────────────────────────────────────
  function getInputs() {
    const n = id => parseFloat(document.getElementById(id).value);
    return {
      Nx:      Math.max(3, parseInt(document.getElementById('app1-nx').value) || 20),
      Ny:      Math.max(3, parseInt(document.getElementById('app1-ny').value) || 20),
      Lx:      n('app1-lx') || 1.0,
      Ly:      n('app1-ly') || 1.0,
      S:       isNaN(n('app1-s'))       ? 0 : n('app1-s'),
      Tleft:   isNaN(n('app1-t-left'))  ? 0 : n('app1-t-left'),
      Tright:  isNaN(n('app1-t-right')) ? 0 : n('app1-t-right'),
      Ttop:    isNaN(n('app1-t-top'))   ? 0 : n('app1-t-top'),
      Tbottom: isNaN(n('app1-t-bottom'))? 0 : n('app1-t-bottom'),
    };
  }

  function buildGrid(Nx, Ny, Lx, Ly) {
    const dx = Lx / (Nx - 1), dy = Ly / (Ny - 1);
    const xs = [], ys = [];
    for (let i = 0; i < Nx; i++) xs.push(i * dx);
    for (let j = 0; j < Ny; j++) ys.push(j * dy);
    return { xs, ys, dx, dy };
  }

  // ── Plot layout (theme-neutral transparent background) ─────────────────────
  const AXIS = {
    zeroline: false,
    color: '#999',
    gridcolor: 'rgba(150,150,150,0.15)',
    linecolor: 'rgba(150,150,150,0.3)',
  };

  function plotLayout(extra) {
    return Object.assign({
      paper_bgcolor: 'rgba(0,0,0,0)',
      plot_bgcolor:  'rgba(0,0,0,0)',
      font: { color: '#888', size: 11 },
      margin: { t: 48, l: 52, r: 24, b: 52 },
      hoverlabel: { bgcolor: '#1e293b', font: { color: '#f1f5f9', size: 12 } },
    }, extra);
  }

  // ── Live badge ──────────────────────────────────────────────────────────────
  function setBadge(state) {
    const b = document.getElementById('app1-live-badge');
    const l = document.getElementById('app1-live-label');
    if (!b) return;
    b.className = state ? `active ${state}` : 'active';
    const labels = { pending: 'En attente…', solving: 'Calcul…', '': 'En direct' };
    if (l) l.textContent = labels[state] || 'En direct';
  }

  // ── Auto-solve debounce ─────────────────────────────────────────────────────
  let debounceTimer = null;
  function scheduleAutoSolve() {
    setBadge('pending');
    clearTimeout(debounceTimer);
    debounceTimer = setTimeout(app1Solve, 380);
  }

  // Attach input listeners + run initial solve after page is ready
  setTimeout(function () {
    ['app1-nx','app1-ny','app1-lx','app1-ly','app1-s',
     'app1-t-top','app1-t-left','app1-t-right','app1-t-bottom'].forEach(function (id) {
      const el = document.getElementById(id);
      if (el) el.addEventListener('input', scheduleAutoSolve);
    });
    app1Solve();
  }, 80);

  // ── Mesh plot ───────────────────────────────────────────────────────────────
  window.app1ShowMesh = function () {
    const { Nx, Ny, Lx, Ly } = getInputs();
    const { xs, ys } = buildGrid(Nx, Ny, Lx, Ly);

    const gx = [], gy = [];
    for (let j = 0; j < Ny; j++) { gx.push(xs[0], xs[Nx-1], null); gy.push(ys[j], ys[j], null); }
    for (let i = 0; i < Nx; i++) { gx.push(xs[i], xs[i], null);    gy.push(ys[0], ys[Ny-1], null); }

    Plotly.react('app1-mesh-plot', [
      { x: gx, y: gy, mode: 'lines', type: 'scatter',
        line: { color: 'rgba(99,102,241,0.45)', width: 0.8 },
        hoverinfo: 'skip', showlegend: false },
      { x: [xs[0], xs[Nx-1], xs[Nx-1], xs[0], xs[0]],
        y: [ys[0], ys[0],    ys[Ny-1], ys[Ny-1], ys[0]],
        mode: 'lines', type: 'scatter',
        line: { color: '#f97316', width: 2.5 },
        hoverinfo: 'skip', showlegend: false },
    ], plotLayout({
      title: { text: `Maillage — ${Nx} × ${Ny} nœuds`, font: { size: 13 } },
      xaxis: Object.assign({}, AXIS, { title: { text: 'x' } }),
      yaxis: Object.assign({}, AXIS, { title: { text: 'y' }, scaleanchor: 'x', scaleratio: 1 }),
    }), { responsive: true });
  };

  // ── FD solver (Gauss-Seidel) ────────────────────────────────────────────────
  function gaussSeidel(Nx, Ny, dx, dy, S, bc) {
    const T = new Float64Array(Nx * Ny);
    const idx = (i, j) => j * Nx + i;

    // Apply Dirichlet boundary conditions
    for (let i = 0; i < Nx; i++) {
      T[idx(i, 0)]      = bc.Tbottom;
      T[idx(i, Ny - 1)] = bc.Ttop;
    }
    for (let j = 0; j < Ny; j++) {
      T[idx(0, j)]      = bc.Tleft;
      T[idx(Nx - 1, j)] = bc.Tright;
    }
    // Corners: average of the two adjacent boundaries
    T[idx(0,      0)]      = (bc.Tleft  + bc.Tbottom) / 2;
    T[idx(Nx - 1, 0)]      = (bc.Tright + bc.Tbottom) / 2;
    T[idx(0,      Ny - 1)] = (bc.Tleft  + bc.Ttop)    / 2;
    T[idx(Nx - 1, Ny - 1)] = (bc.Tright + bc.Ttop)    / 2;

    const invDx2 = 1 / (dx * dx), invDy2 = 1 / (dy * dy);
    const denom  = 2 * (invDx2 + invDy2);
    let iter = 0, residual = Infinity;

    while (iter < 10000 && residual > 1e-6) {
      residual = 0;
      for (let j = 1; j <= Ny - 2; j++) {
        for (let i = 1; i <= Nx - 2; i++) {
          // Residual uses the OLD T[i,j] before update (otherwise it is
          // trivially zero since Gauss-Seidel solves the local stencil exactly)
          const res = Math.abs(
            (T[idx(i-1,j)] - 2*T[idx(i,j)] + T[idx(i+1,j)]) * invDx2 +
            (T[idx(i,j-1)] - 2*T[idx(i,j)] + T[idx(i,j+1)]) * invDy2 + S
          );
          if (res > residual) residual = res;
          T[idx(i, j)] = (
            (T[idx(i-1,j)] + T[idx(i+1,j)]) * invDx2 +
            (T[idx(i,j-1)] + T[idx(i,j+1)]) * invDy2 + S
          ) / denom;
        }
      }
      iter++;
    }
    return { T, iter, residual };
  }

  // ── Solve & render ──────────────────────────────────────────────────────────
  window.app1Solve = function () {
    const { Nx, Ny, Lx, Ly, S, Tleft, Tright, Ttop, Tbottom } = getInputs();
    const { xs, ys, dx, dy } = buildGrid(Nx, Ny, Lx, Ly);

    setBadge('solving');
    document.getElementById('app1-status').textContent = 'Calcul en cours…';

    // Yield to browser so the badge update is visible before the blocking solve
    setTimeout(function () {
      const { T, iter, residual } = gaussSeidel(Nx, Ny, dx, dy, S, { Tleft, Tright, Ttop, Tbottom });

      const z = [];
      for (let j = 0; j < Ny; j++) {
        const row = [];
        for (let i = 0; i < Nx; i++) row.push(T[j * Nx + i]);
        z.push(row);
      }

      Plotly.react('app1-solution-plot', [
        { type: 'contour', z, x: xs, y: ys,
          colorscale: 'Turbo',
          contours: { coloring: 'heatmap', showlines: true },
          line: { color: 'rgba(255,255,255,0.38)', width: 0.8 },
          ncontours: 20,
          zsmooth: 'best',
          colorbar: {
            title: { text: 'T', side: 'right' },
            tickfont: { size: 10 },
            thickness: 13, len: 0.88,
            bgcolor: 'rgba(0,0,0,0)',
          },
          hovertemplate: 'x=%{x:.3f}<br>y=%{y:.3f}<br>T=%{z:.4f}<extra></extra>',
        },
      ], plotLayout({
        title: { text: 'Champ de température T(x, y)', font: { size: 13 } },
        xaxis: Object.assign({}, AXIS, { title: { text: 'x' } }),
        yaxis: Object.assign({}, AXIS, { title: { text: 'y' }, scaleanchor: 'x', scaleratio: 1 }),
        margin: { t: 48, l: 52, r: 80, b: 52 },
      }), { responsive: true });

      const Tvals = Array.from(T);
      const Tmax = Math.max(...Tvals), Tmin = Math.min(...Tvals);
      const ok = residual <= 1e-6;
      document.getElementById('app1-status').textContent = ok
        ? `✓  Convergé en ${iter} itérations — T ∈ [${Tmin.toFixed(3)}, ${Tmax.toFixed(3)}]`
        : `⚠  ${iter} itérations (résidu ${residual.toExponential(1)}) — T ∈ [${Tmin.toFixed(3)}, ${Tmax.toFixed(3)}]`;

      setBadge('');
      app1ShowMesh();
    }, 20);
  };

})();
</script>
