# Imperial_Executive_Education_Capstone2026
Capstone project developed for the Imperial College Business School **Professional Certificate in Machine Learning and Artificial Intelligence**.

# Bayesian Black-Box Optimisation — Capstone Project

![Status](https://img.shields.io/badge/Status-COMPLETE-brightgreen)
![Python](https://img.shields.io/badge/Python-3.8%2B-blue)
![Type](https://img.shields.io/badge/Type-Capstone-green)
![Modules](https://img.shields.io/badge/Modules-12--24-purple)
![Week](https://img.shields.io/badge/All%2013%20Weeks-Done-brightgreen)

Over 13 rounds, this project optimises **eight unknown black-box functions** of increasing dimensionality (2D to 8D) under a strict query budget, one new query per function per round. The internal equations are unknown and only inputs and outputs are observable, resembling real-world problems where the underlying model cannot be ascertained. The task is to find the input that **maximises** each function through disciplined, evidence-driven, iterative decisions.

The exercise mirrors how optimisation actually works in research and industry — hyperparameter tuning, experimental design, process optimisation, drug discovery — where evaluations are expensive, data is scarce, and the system is a black box.

---

## Documentation

| Document | What it covers |
|----------|----------------|
| [STRATEGY & APPROACH](BBO_Strategy_and_Approach.md) | Core method, pipeline, strategy evolution, final results, lessons learned |
| [MODEL CARD](BBO_Modelcard.md) | The optimisation approach, performance, assumptions, limitations, ethics |
| [DATASHEET](BBO_Datasheet.md) | Dataset composition, collection process, preprocessing, intended and inappropriate uses |
| [`initial_data/`](initial_data) | Seed observations (`.npy`) provided as the starting point for each function |

---

## The approach in brief

Each round, for every function, a uniform pipeline runs:

1. **Fit two surrogates** — a tuned Gaussian Process (primary) and an SVR (independent cross-check).
2. **Diagnose locally** — a per-dimension "nudge" test maps the gradient around the current best, independent of the acquisition function.
3. **Propose candidates** — the expected-improvement point, the strongest nudge directions, and a manual hypothesis-driven point.
4. **Filter with the "untrust rule"** — never chase an acquisition candidate the model itself does not rate above the current best.
5. **Decide** — submit the best-supported candidate, overriding the model when the diagnostic gives clearer evidence, and log the full rationale.

In the final weeks, **K-means clustering** was added as a diagnostic to characterise each function's search space — separating high-value regions from exploration scatter, confirming multimodality, and exposing dimensions that had never been varied inside the best region.

Full detail: [Strategy & Approach](BBO_Strategy_&_Approach.md).

---

## Functions

| Function | Dim | Scenario | Objective |
|----------|-----|----------|-----------|
| F1 | 2D | Radiation Detection | Locate a contamination source in a 2D field |
| F2 | 2D | Noisy Model | Maximise a noisy log-likelihood |
| F3 | 3D | Drug Discovery | Minimise side effects (negated for maximisation) |
| F4 | 4D | Warehouse Placement | Optimise product placement |
| F5 | 4D | Chemical Yield | Maximise yield of a unimodal process |
| F6 | 5D | Cake Recipe Optimization | Maximise a recipe score (negative by design, pushed toward zero) |
| F7 | 6D | ML Hyperparameter Tuning | Maximise model accuracy / F1 |
| F8 | 8D | 8D Optimisation | Maximise validation accuracy across eight hyperparameters |

**Inputs**: numerical vectors with every value in [0, 1], submitted as a
hyphen-separated string to six decimals (e.g. `0.241041-0.805036-0.905090`).
**Outputs**: a single scalar — higher is better.
**Seed data**: 10 to 40 observations per function (varying by function), growing by one per round.

---

## Results

Best output found per function over the campaign:

| Function | Dim | Best output | Notes |
|----------|-----|-------------|-------|
| F1 | 2D | **9.24e-5** | Narrow ridge navigated by small disciplined steps; a per-dimension override found the active axis after a long gradient exhausted, and the final round added a further **+37%** |
| F2 | 2D | **0.7184** | An exploration probe beat local refinement; function confirmed **bimodal** |
| F3 | 3D | **-0.0302** | Converged early to a narrow single peak |
| F4 | 4D | **0.7420** | Tight bracketed peak; clustering revealed untouched dimensions inside the best region |
| F5 | 4D | **8662.48** | Boundary push gave the biggest single-round gain (**+85%**); a strongly nonlinear ridge, invisible to the surrogate, surfaced via clustering |
| F6 | 5D | **-0.1609** | The most volatile function (GP error ~67%); minimal steps and distrusting the model finally beat a long-standing best in the last round |
| F7 | 6D | **2.9906** | Two bold, well-calibrated multi-dimensional moves the surrogate under-predicted — the final round added **+39%** on top of an earlier +19% |
| F8 | 8D | **9.9345** | Converged; model confidence kept rising while real gains decayed to noise |

Numbers reflect the final results across all 13 rounds; see the
[Model Card](BBO_Modelcard.md) for the full performance discussion.

---

## What this project demonstrates

This capstone is a compact demonstration of skills that transfer directly to data science, ML engineering, and quantitative decision-making roles:

- **Decision-making under uncertainty** — every round committed to one action on incomplete information, with an explicit, logged rationale.
- **Working with scarce, expensive data** — the entire campaign ran on tens of data points per function, exactly the regime where careful modelling matters most (clinical, experimental, and process data rarely arrive in abundance).
- **Surrogate modelling and calibration** — fitting Gaussian Processes and SVRs, reading their calibration honestly, and knowing when *not* to trust a confident model.
- **Diagnostic rigour** — building independent checks (per-dimension nudge tests, dual surrogates, clustering) rather than trusting a single black-box recommendation.
- **Communication and documentation** — a datasheet, a model card, and round-by-round reflections that make every decision auditable and reproducible.

These are the same competencies behind model tuning, experimental design, A/B testing, and any setting where a system must be improved without full knowledge of how it works — including data-driven innovation in health and MedTech.

---

## Repository structure

```
Imperial_Executive_Education_Capstone2026/
│
├── README.md
├── BBO_Strategy_and_Approach.md
├── BBO_Modelcard.md
├── BBO_Datasheet.md
│
├── F1_Radiation_Detection/
├── F2_Noisy_Model/
├── F3_Drug_Discovery/
├── F4_Warehouse_Placement/
├── F5_Chemical_Yield/
├── F6_Cake_Recipe_Optimization/
├── F7_ML_Hyperparameter_Tuning/
├── F8_8D_Optimization/
│     (each function folder contains its per-round query pipeline,
│      K-means analysis notebook, and per-round log)
│
├── Weekly_Reflection/     ← round-by-round written reflections
└── initial_data/          ← seed observations (.npy) for all functions
```

---

## Technologies used

| Category | Technologies |
|----------|-------------|
| Language | Python |
| Libraries | NumPy, SciPy, scikit-learn, Matplotlib |
| Models | Gaussian Process (Matérn / RBF kernels), SVR |
| Diagnostics | Per-dimension nudge test, K-means clustering (with silhouette selection), PCA for visualisation |
| Acquisition | Expected Improvement (bounded radius) |
| Tooling | Jupyter Notebooks, Git, GitHub |

---

## How to run

```bash
# Clone
git clone https://github.com/GPAjay/Imperial_Executive_Education_Capstone2026.git
cd Imperial_Executive_Education_Capstone2026

# Environment
pip install numpy scipy scikit-learn matplotlib jupyter

# Open any function's pipeline or K-means notebook
jupyter notebook F1_Radiation_Detection/
```

Each function folder is self-contained: run its pipeline to reproduce a
round's candidate analysis, or open its K-means notebook to reproduce
the clustering diagnostics.

---

## Author

**Ajay Kumar Ginka Panasa** — Imperial College Business School AI/ML
Programme.

🔗 [GitHub](https://github.com/GPAjay/Imperial_Executive_Education_Capstone2026)
