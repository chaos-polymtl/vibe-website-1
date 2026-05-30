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
#app1-inputs {
  display: flex;
  flex-wrap: wrap;
  gap: 1.2rem;
  margin-bottom: 1rem;
  align-items: flex-end;
}
#app1-inputs label {
  display: flex;
  flex-direction: column;
  font-size: 0.85rem;
  gap: 0.3rem;
}
#app1-inputs input[type="number"] {
  width: 90px;
  padding: 0.3rem 0.5rem;
  border: 1px solid var(--md-default-fg-color--lighter);
  border-radius: 4px;
  background: var(--md-default-bg-color);
  color: var(--md-default-fg-color);
  font-size: 0.9rem;
}
.app1-btn {
  background: var(--md-primary-fg-color);
  color: var(--md-primary-bg-color);
  border: none;
  padding: 0.5rem 1.4rem;
  border-radius: 4px;
  cursor: pointer;
  font-size: 0.9rem;
  margin-right: 0.5rem;
  margin-bottom: 0.5rem;
}
.app1-btn:hover {
  opacity: 0.85;
}
#app1-status {
  font-size: 0.85rem;
  color: var(--md-default-fg-color--light);
  margin: 0.5rem 0;
  min-height: 1.2em;
}
#app1-plots {
  display: flex;
  flex-wrap: wrap;
  gap: 1rem;
  margin-top: 0.5rem;
}
#app1-mesh-plot,
#app1-solution-plot {
  min-width: 350px;
  flex: 1;
  min-height: 380px;
}
#app1-bc {
  margin: 0.8rem 0;
}
#app1-bc-title {
  font-size: 0.85rem;
  font-weight: 600;
  margin-bottom: 0.5rem;
  color: var(--md-default-fg-color);
}
#app1-bc-grid {
  display: grid;
  grid-template-columns: auto auto auto;
  grid-template-rows: auto auto auto;
  gap: 0.4rem 0.6rem;
  align-items: center;
  justify-items: center;
  width: fit-content;
}
#app1-bc-grid label {
  display: flex;
  flex-direction: column;
  font-size: 0.8rem;
  gap: 0.2rem;
  align-items: center;
}
#app1-bc-grid input[type="number"] {
  width: 75px;
  padding: 0.25rem 0.4rem;
  border: 1px solid var(--md-default-fg-color--lighter);
  border-radius: 4px;
  background: var(--md-default-bg-color);
  color: var(--md-default-fg-color);
  font-size: 0.85rem;
  text-align: center;
}
#app1-bc-domain {
  width: 80px;
  height: 48px;
  border: 1px dashed var(--md-default-fg-color--lighter);
  border-radius: 4px;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 0.75rem;
  color: var(--md-default-fg-color--light);
}
</style>

<div id="app1-container">

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

  <div id="app1-bc">
    <div id="app1-bc-title">Conditions aux limites (Dirichlet)</div>
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

  <button class="app1-btn" onclick="app1ShowMesh()">Afficher le maillage</button>
  <button class="app1-btn" onclick="app1Solve()">Résoudre</button>

  <p id="app1-status"></p>

  <div id="app1-plots">
    <div id="app1-mesh-plot"></div>
    <div id="app1-solution-plot"></div>
  </div>

</div>

<script>
(function () {

  function getInputs() {
    return {
      Nx:      Math.max(3, parseInt(document.getElementById('app1-nx').value) || 20),
      Ny:      Math.max(3, parseInt(document.getElementById('app1-ny').value) || 20),
      Lx:      parseFloat(document.getElementById('app1-lx').value) || 1.0,
      Ly:      parseFloat(document.getElementById('app1-ly').value) || 1.0,
      S:       parseFloat(document.getElementById('app1-s').value)  || 0.0,
      Tleft:   parseFloat(document.getElementById('app1-t-left').value)   || 0.0,
      Tright:  parseFloat(document.getElementById('app1-t-right').value)  || 0.0,
      Ttop:    parseFloat(document.getElementById('app1-t-top').value)    || 0.0,
      Tbottom: parseFloat(document.getElementById('app1-t-bottom').value) || 0.0,
    };
  }

  function buildGrid(Nx, Ny, Lx, Ly) {
    const dx = Lx / (Nx - 1);
    const dy = Ly / (Ny - 1);
    const xs = [], ys = [];
    for (let i = 0; i < Nx; i++) xs.push(i * dx);
    for (let j = 0; j < Ny; j++) ys.push(j * dy);
    return { xs, ys, dx, dy };
  }

  window.app1ShowMesh = function () {
    const { Nx, Ny, Lx, Ly } = getInputs();
    const { xs, ys } = buildGrid(Nx, Ny, Lx, Ly);

    const px = [], py = [];
    for (let j = 0; j < Ny; j++) {
      for (let i = 0; i < Nx; i++) {
        px.push(xs[i]);
        py.push(ys[j]);
      }
    }

    Plotly.react('app1-mesh-plot', [{
      x: px, y: py,
      mode: 'markers',
      type: 'scatter',
      marker: { size: 4, color: 'var(--md-accent-fg-color, teal)' },
      hovertemplate: 'x=%{x:.3f}<br>y=%{y:.3f}<extra></extra>',
    }], {
      title: { text: 'Maillage' },
      xaxis: { title: 'x', zeroline: false },
      yaxis: { title: 'y', zeroline: false, scaleanchor: 'x', scaleratio: 1 },
      margin: { t: 40, l: 50, r: 20, b: 50 },
      paper_bgcolor: 'rgba(0,0,0,0)',
      plot_bgcolor: 'rgba(0,0,0,0)',
      font: { color: getComputedStyle(document.documentElement).getPropertyValue('--md-default-fg-color') || '#333' },
    }, { responsive: true });

    document.getElementById('app1-status').textContent = `Maillage affiché : ${Nx} × ${Ny} = ${Nx * Ny} nœuds`;
  };

  function gaussSeidel(Nx, Ny, dx, dy, S, bc) {
    const T = new Float64Array(Nx * Ny);
    const idx = (i, j) => j * Nx + i;

    // Apply Dirichlet boundary conditions
    for (let i = 0; i < Nx; i++) {
      T[idx(i, 0)]      = bc.Tbottom;  // bottom row (j=0)
      T[idx(i, Ny - 1)] = bc.Ttop;     // top row    (j=Ny-1)
    }
    for (let j = 0; j < Ny; j++) {
      T[idx(0, j)]      = bc.Tleft;    // left col   (i=0)
      T[idx(Nx - 1, j)] = bc.Tright;   // right col  (i=Nx-1)
    }
    // Corners: average of the two adjacent boundaries
    T[idx(0,      0)]      = (bc.Tleft  + bc.Tbottom) / 2;
    T[idx(Nx - 1, 0)]      = (bc.Tright + bc.Tbottom) / 2;
    T[idx(0,      Ny - 1)] = (bc.Tleft  + bc.Ttop)    / 2;
    T[idx(Nx - 1, Ny - 1)] = (bc.Tright + bc.Ttop)    / 2;
    const invDx2 = 1.0 / (dx * dx);
    const invDy2 = 1.0 / (dy * dy);
    const denom  = 2.0 * (invDx2 + invDy2);
    const maxIter = 10000;
    const tol = 1e-6;

    let iter = 0;
    let residual = Infinity;

    while (iter < maxIter && residual > tol) {
      residual = 0.0;

      for (let j = 1; j <= Ny - 2; j++) {
        for (let i = 1; i <= Nx - 2; i++) {
          // Residual uses the OLD T[i,j] before update (otherwise it is
          // trivially zero since Gauss-Seidel solves the local stencil exactly)
          const res = Math.abs(
            (T[idx(i-1,j)] - 2*T[idx(i,j)] + T[idx(i+1,j)]) * invDx2 +
            (T[idx(i,j-1)] - 2*T[idx(i,j)] + T[idx(i,j+1)]) * invDy2 +
            S
          );
          if (res > residual) residual = res;

          T[idx(i, j)] = (
            (T[idx(i - 1, j)] + T[idx(i + 1, j)]) * invDx2 +
            (T[idx(i, j - 1)] + T[idx(i, j + 1)]) * invDy2 +
            S
          ) / denom;
        }
      }
      iter++;
    }

    return { T, iter, residual };
  }

  window.app1Solve = function () {
    const { Nx, Ny, Lx, Ly, S, Tleft, Tright, Ttop, Tbottom } = getInputs();
    const { xs, ys, dx, dy } = buildGrid(Nx, Ny, Lx, Ly);

    document.getElementById('app1-status').textContent = 'Résolution en cours…';

    // Yield to browser to paint the status message before the blocking solve
    setTimeout(function () {
      const { T, iter, residual } = gaussSeidel(Nx, Ny, dx, dy, S, { Tleft, Tright, Ttop, Tbottom });

      // Build 2D z array: z[j][i], j=0 is bottom (y=0)
      const z = [];
      for (let j = 0; j < Ny; j++) {
        const row = [];
        for (let i = 0; i < Nx; i++) row.push(T[j * Nx + i]);
        z.push(row);
      }

      Plotly.react('app1-solution-plot', [{
        type: 'heatmap',
        z: z,
        x: xs,
        y: ys,
        colorscale: 'Viridis',
        colorbar: { title: { text: 'T', side: 'right' } },
        hovertemplate: 'x=%{x:.3f}<br>y=%{y:.3f}<br>T=%{z:.5f}<extra></extra>',
      }], {
        title: { text: 'Champ de température' },
        xaxis: { title: 'x' },
        yaxis: { title: 'y', scaleanchor: 'x', scaleratio: 1 },
        margin: { t: 40, l: 50, r: 80, b: 50 },
        paper_bgcolor: 'rgba(0,0,0,0)',
        plot_bgcolor: 'rgba(0,0,0,0)',
        font: { color: getComputedStyle(document.documentElement).getPropertyValue('--md-default-fg-color') || '#333' },
      }, { responsive: true });

      const converged = residual <= 1e-6;
      const Tmax = Math.max(...Array.from(T));
      document.getElementById('app1-status').textContent =
        converged
          ? `Convergé en ${iter} itérations (résidu = ${residual.toExponential(2)}). T_max = ${Tmax.toFixed(5)}`
          : `Limite de ${iter} itérations atteinte (résidu = ${residual.toExponential(2)}). T_max = ${Tmax.toFixed(5)}`;

      // Also refresh the mesh plot so both panels are visible
      app1ShowMesh();
    }, 20);
  };

})();
</script>
