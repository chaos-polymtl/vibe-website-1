---
hide:
  - navigation
---

<h1 style="display: flex; flex-wrap: wrap; align-items: center; justify-content: space-between;">
  <span>Outils pour l'enseignement du génie chimique</span>
  <span style="display: flex; gap: 0.5rem; flex-shrink: 0;">
  <img
    src="assets/chaos_logo_black_without_bkg.png"
    alt="CHAOS logo"
    style="height:80px;margin-left:1rem;"
    class="chaos-logo">
  </span>
</h1>

<style>
/* Hide/show the right logo according to the active Material theme */
[data-md-color-scheme="slate"] .chaos-logo {
  content: url("assets/chaos_logo_white_without_bkg.png");
}
[data-md-color-scheme="default"] .chaos-logo {
  content: url("assets/chaos_logo_black_without_bkg.png");
}
</style>



Cette page regroupe certains outils servant à l'enseignement du génie chimique.

---

<div class="grid cards" markdown>

-   :material-thermometer:{ .lg .middle } **Solveur de chaleur 2D**

    ---

    Résolution de l'équation de Poisson (∇²T = −S) par différences finies sur un domaine rectangulaire avec conditions aux limites de Dirichlet.

    [:octicons-arrow-right-24: Lancer l'application](apps/application-1.md)

-   :material-water-pump:{ .lg .middle } **Contrôle de niveau d'un réservoir**

    ---

    Simulation dynamique d'un réservoir dont le niveau est régulé par un régulateur PI agissant sur la vanne de sortie. Dynamique des procédés et contrôle.

    [:octicons-arrow-right-24: Lancer l'application](apps/application-2.md)

-   :material-molecule:{ .lg .middle } **Application 3**

    ---

    Description de la troisième application pédagogique. À venir.

    [:octicons-arrow-right-24: En savoir plus](apps/application-3.md)

</div>
