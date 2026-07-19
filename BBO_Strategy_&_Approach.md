# Strategy & Approach

How eight unknown functions were optimised over 13 rounds: the core
method, how it evolved, the per-function pipeline, and the lessons that
shaped each decision.

---

## Core method

- **Surrogate models**: a **Gaussian Process** (primary) cross-checked
  by an **SVR** (secondary). Using two independent surrogates exposed
  cases where one model was structurally unreliable.
- **GP tuning**: kernel-and-alpha grid search over Matérn
  (ν = 0.5 / 1.5 / 2.5) and RBF kernels, alpha from 1e-8 to 1e-1,
  selected by log marginal likelihood, with per-dimension length scales.
- **Acquisition**: bounded-radius **Expected Improvement** around the
  current best.
- **Per-dimension nudge diagnostic**: small symmetric perturbations in
  each dimension to map the local gradient independently of the
  acquisition function.
- **Calibration-trust rule**: how far the surrogate is trusted is tied
  to how accurately it predicts the value at the *current best*.
- **Clustering diagnostics** (late campaign): K-means over each
  function's queried points to characterise the search space.

## How the pipeline works

Each round, per function:

1. **Load data** — stack the seed observations with all prior
   submissions and their returned values.
2. **Transform outputs** — apply symlog10 to functions whose outputs
   span orders of magnitude (to stabilise the GP fit); use raw values
   for functions with narrow, well-behaved ranges.
3. **Tune the GP** — grid-search kernel and alpha, pick the highest log
   marginal likelihood, read off the per-dimension length scales.
4. **Tune the SVR** — grid-search C / gamma / epsilon as an independent
   cross-check.
5. **Run the per-dimension nudge test** — perturb the current best in
   each dimension (±small steps) and record predicted change.
6. **Optimise EI** — search for the expected-improvement-maximising
   point within a bounded radius of the current best.
7. **Assemble candidates** — the EI point, the strongest nudge
   directions, and a manual candidate encoding the round's hypothesis.
8. **Apply the untrust rule** — discard any acquisition candidate whose
   GP mean prediction sits *below* the current best (improvement driven
   only by uncertainty, not by predicted value).
9. **Decide** — submit the best-supported candidate, overriding the
   model when the diagnostic gives clearer evidence than the
   acquisition function.
10. **Log** — record every candidate, both surrogates' predictions,
    calibration error at the best, and the chosen submission.

## Key techniques

- **Dual surrogates (GP + SVR)** to catch structural model failure —
  SVR consistently hallucinated on the widest-range functions, which
  was itself a useful signal.
- **symlog10 transforms** for functions with extreme output ranges;
  raw values otherwise.
- **Per-dimension nudge tests** as the workhorse diagnostic — they
  repeatedly surfaced winning directions the acquisition function
  missed.
- **Calibration-trust scaling** — boldness scaled inversely with GP
  error at the current best: near-perfect calibration licensed bold
  multi-dimensional moves; poor calibration forced minimal single-step
  probes.
- **The untrust rule** — never chase an acquisition candidate the model
  itself does not rate above the current best.
- **K-means clustering diagnostics** — per-function, to separate
  high-value regions from exploration scatter, confirm multimodality,
  and expose dimensions never varied inside the best region.
- **Per-function policies** — each function was handled according to its
  regime rather than by a single global rule.

## Strategy evolution

**Exploration (early rounds).** Broad, acquisition-driven search from
the seed data. Little specialisation; several functions still flat or
negative.

**Regime detection (middle rounds).** The eight functions clearly
behaved differently — some smooth and monotone, some noisy, some
multimodal, one highly volatile. Per-function step sizes, candidate
patterns, and override logic emerged here.

**Model–data interplay and clustering (late rounds).** The per-dimension
nudge diagnostic and the untrust rule matured into a consistent
decision procedure, and K-means clustering was added to reason about the
search space geometry directly rather than through the surrogate alone.

## Final per-function results

Best output found per function over the campaign (all functions are
maximisation tasks; F3 and F6 are negative by design and pushed toward
zero):

| Function | Dim | Best output | Found in | What worked |
|----------|-----|-------------|----------|-------------|
| F1 Radiation Detection | 2D | **9.24e-5** | final round | Small disciplined steps down a narrow ridge; a per-dimension override caught the active axis after a long gradient exhausted, and the final round added a further **+37%** |
| F2 Noisy Model | 2D | **0.7184** | mid | A deliberate exploration probe into an empty region beat many rounds of local refinement; function later confirmed **bimodal** |
| F3 Drug Discovery | 3D | **-0.0302** | early | Converged early to a narrow single peak; later rounds characterised the peak rather than beating it |
| F4 Warehouse Placement | 4D | **0.7420** | early | Tight bracketed peak; clustering later revealed two dimensions were never varied inside the best region |
| F5 Chemical Yield | 4D | **8662.48** | mid | Boundary push to the corner gave the campaign's biggest single-round gain (**+85%**); a strongly nonlinear ridge, invisible to the surrogate, surfaced via clustering |
| F6 Cake Recipe | 5D | **-0.1609** | final round | The project's most volatile function (GP calibration error ~67%); minimal steps and distrusting the model finally beat a best that had stood since early in the campaign (**+13.5%**) |
| F7 Hyperparameter Tuning | 6D | **2.9906** | final round | Two bold, well-calibrated multi-dimensional acquisition moves that the surrogate *under*-predicted — a **+19%** mid-campaign gain followed by a **+39%** gain in the final round |
| F8 8D Optimisation | 8D | **9.9345** | final round | Converged single peak; model confidence (log marginal likelihood) kept climbing while real gains decayed to noise — final-round improvement was within rounding |

## Lessons learned

| Theme | Lesson |
|-------|--------|
| Exploration pays | On one function, a single exploration probe into an empty region beat rounds of local refinement — direct evidence the search was exploitation-biased and that untested regions can hold better optima. |
| Trust ≠ calibration everywhere | A surrogate can be near-perfectly calibrated at the current best and still extrapolate badly nearby. Local calibration is local information, not a global guarantee. |
| Boldness scales with calibration | Where calibration was excellent, bold multi-dimensional moves produced the biggest emergent wins. Where it was poor, the same boldness was catastrophic. |
| Gradients exhaust | A run of successful steps in one direction does not mean the next step succeeds. The per-dimension diagnostic reliably flagged when a gradient had run out and a different axis had become active. |
| Cross-check surrogates | A second surrogate (SVR) caught structural model failure the GP alone would have hidden — on the widest-range functions SVR predictions were nonsense, and knowing that mattered. |
| Confidence ≠ headroom | One function's model grew steadily more confident (rising log marginal likelihood) while its actual improvements decayed to noise. Trusting the model includes trusting it when it says to stop. |
| Clustering exposes blind spots | K-means made plain what the surrogate could not: which dimensions had never been varied inside the best region, and whether a "cluster" was a genuine converged peak or a volatile region masquerading as one. |
| Volatility needs its own metric | Spatial clustering alone cannot tell a converged peak from a noisy one; an output-volatility measure (change in output per unit movement in input) was needed to separate signal from noise. |
| A validated pattern repeats | The same well-calibrated, bold multi-dimensional move that first opened a new regime on one function paid off *again* in the final round, adding a further large gain. Once a pattern is validated by evidence, re-applying it deliberately is more than luck. |
| Patience beats forcing it | On the most volatile function, a best had stood since early in the campaign. Restraint — minimal steps, no forcing — rather than aggressive probing was what eventually beat it in the last round. |

## Technologies used

| Category | Technologies |
|----------|-------------|
| Language | Python |
| Libraries | NumPy, SciPy, scikit-learn, Matplotlib |
| Models | Gaussian Process (Matérn / RBF kernels), SVR |
| Diagnostics | Per-dimension nudge test, K-means clustering (with silhouette selection), PCA for visualisation |
| Acquisition | Expected Improvement (bounded radius) |
| Tooling | Jupyter Notebooks, Git, GitHub |
