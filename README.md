# Imperial_Executive_Education_Capstone2026
Capstone project developed for the Imperial College Business School **Professional Certificate in Machine Learning and Artificial Intelligence**.

# Bayesian Black-Box Optimisation (BBO) Challenge:

![Type](https://img.shields.io/badge/Type-Capstone-green)
![Python](https://img.shields.io/badge/Python-3.8%2B-blue)
![Week](https://img.shields.io/badge/Duration-13%20Weeks-brightgreen)
![Modules](https://img.shields.io/badge/Modules-12--24-purple)
![Status](https://img.shields.io/badge/Status-COMPLETE-brightgreen)

This project optimises **eight unknown black-box functions** of increasing dimensionality (2D to 8D) under a strict query budget, where we get one new query per function per round (13 weeks). The internal equations are unknown, and only the inputs and outputs are observable. This setup resembles real-world problems where the underlying model cannot be ascertained, and data is often limited. The task is to find the input that **maximises** each function through iterative, evidence-driven decisions.

The capstone project illustrates how black-box optimisation is applied across research and industry. Examples include ML hyperparameter tuning, experimental design, process optimisation, and drug discovery. In these real-world scenarios, evaluations are often expensive in both cost and time, data is scarce, and the underlying system is a black box. Working with a limited number of query points, the aim is to map the function's landscape and locate its maximum, thereby demonstrating that a black-box function can be characterised and optimised effectively from relatively few samples.

## Career relevance: what this project demonstrates

This capstone is a compact demonstration of skills that transfer directly to data science, ML engineering, and quantitative decision-making roles:

- **Decision-making under uncertainty:** Each query point is committed to an action/decision based on incomplete information.
- **Working with scarce, expensive data:** The project starts with a few data points, presenting the challenges where careful modelling is most critical (clinical, experimental, and process data are rarely available in abundance).
- **Surrogate modelling and calibration:** Fitting Gaussian Processes and Support Vector Regression (SVRs), decision trees, ensemble modelling, rationalizing the calibration, and developing intuition on when to trust a confident model.
- **Diagnostic rigour:** Building independent diagnostic checks (nudge tests, surrogates, clustering) rather than trusting a single black-box recommendation.
- **Communication and documentation:** A datasheet, a model card, and reflections to make every decision auditable and reproducible.

These are core competencies required for model tuning, experimental design, A/B testing, and any setting where a system must be improved without full knowledge of how it works, including data-driven innovation in healthcare and MedTech.

## Documentation

| Document | What it covers |
|----------|----------------|
| [DATASHEET](BBO_Datasheet.md) | Dataset composition, collection process, preprocessing, intended and inappropriate uses |
| [MODEL CARD](BBO_Modelcard.md) | The optimisation approach, performance, assumptions, limitations, ethics |
| [STRATEGY & APPROACH](BBO_Strategy_and_Approach.md) | Core method, pipeline, strategy evolution, final results, lessons learned |
| [`initial_data/`](initial_data) | Seed observations (`X/Y.npy`) provided as the starting point for each function |

## Functions

| Function | Dim | Scenario | Objective |
|----------|-----|----------|-----------|
| F1 | 2D | Radiation Detection | Locate a contamination source in a 2D field |
| F2 | 2D | Noisy Model | Maximise a noisy log-likelihood |
| F3 | 3D | Drug Discovery | Minimise side effects of new drug (negated for maximisation) |
| F4 | 4D | Warehouse Placement | Optimise product placement in a warehouse |
| F5 | 4D | Chemical Yield | Maximise yield of a unimodal process |
| F6 | 5D | Cake Recipe Optimization | Maximise a recipe score (negative by design, pushed toward zero) |
| F7 | 6D | ML Hyperparameter Tuning | Maximise model accuracy / F1 |
| F8 | 8D | 8D Optimisation | Maximise validation accuracy across eight hyperparameters |

**Inputs**: numerical vectors with every value in [0, 1], submitted as a hyphen-separated string to six decimals (e.g. `0.241041-0.805036-0.905090`).
**Outputs**: a single scalar value (maxima), the highest positive value or closest to zero for negative data points.
**Seed data**: 10 to 40 observations per function (varying by function), each dataset grows by one per week.

## The approach in brief

A uniform pipeline is designed, which runs for each round for every function:

1. **Fit two surrogates:** a tuned Gaussian Process (primary) and an SVR (independent cross-check).
2. **Diagnose locally:** a per-dimension "nudge" test maps the gradient around the current best, independent of the acquisition function.
3. **Propose candidates:** — Exploration vs Exploitation: expected-improvement (EI) point, the strongest nudge directions, and a manual hypothesis-driven point.
4. **Filter with the untrust rule:** — discard an acquisition candidate that the model itself does not rate above the current best, based on the concept of Reinforcement Learning.
5. **Decide:** — submit the best-supported candidate, overriding the model when the diagnostic gives clearer evidence.

In the final weeks, **K-means clustering** was added as a diagnostic to characterise each function's search space. It separates high-value regions from exploration scatter, confirms multimodality, and exposes dimensions that were never varied inside the best region.

Full detail: [Strategy & Approach](BBO_Strategy_and_Approach.md).

## Results

Best output found per function over the course of the capstone project:

| Function | Dim | Best output | Notes |
|----------|-----|-------------|-------|
| F1 | 2D | **9.24e-5** | Narrow ridge navigated by small steps; a per-dimension observation found the active axis after a long gradient is exhausted, and the final round improves by **+37%** |
| F2 | 2D | **0.7184** | An exploration probe helps overcome local maxima; function is found to be **bimodal** |
| F3 | 3D | **-0.0302** | Converged early to a narrow single peak |
| F4 | 4D | **0.7420** | Tight peak; clustering revealed unexplored dimensions inside the best region |
| F5 | 4D | **8662.48** | Boundary exploration gave the biggest gain (**+85%**); a strongly nonlinear ridge, invisible to the surrogate, surfaced via clustering |
| F6 | 5D | **-0.1609** | The most volatile function (GP error ~67%); minimal steps and distrusting the model under-confidence helps find the best value in the last round |
| F7 | 6D | **2.9906** | Two exploratory, well-calibrated multi-dimensional step identifies the surrogate under-predicted; final round added **+39%** over the previous best |
| F8 | 8D | **9.9345** | Model converged; surrogate confidence improves while real gains were marginal (attributed to noise) |

Numbers reflect the final results across all 13 rounds; see the
[Model Card](BBO_Modelcard.md) for the full performance discussion.

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
│     (each function folder contains its query pipeline, final solution, candidate selection,
│      K-means analysis notebook)
│
├── Weekly_Reflection/     ← Reflections/code of different ML concepts learned throughout the course
└── initial_data/          ← seed observations (X/Y.npy) for all functions
```

## Technologies used

| Category | Technologies |
|----------|-------------|
| Language | Python |
| Libraries | NumPy, SciPy, scikit-learn, Matplotlib |
| Models | Gaussian Process (Matérn / RBF kernels), SVR |
| Diagnostics | Per-dimension nudge test, K-means clustering (with silhouette selection), PCA for visualisation |
| Acquisition | Expected Improvement (bounded radius), Upper Confidence Bound (UCB) |
| Tooling | Jupyter Notebooks, Git, GitHub |

## How to run

```bash
# Clone
git clone https://github.com/GPAjay/Imperial_Executive_Education_Capstone2026.git
cd Imperial_Executive_Education_Capstone2026

# Environment
pip install numpy scipy scikit-learn matplotlib jupyter

# Open any function's pipeline or K-means notebook upyter notebook F1_Radiation_Detection/
```

Each function folder is self-contained: run its pipeline to reproduce a round's candidate analysis, or open its K-means notebook to reproduce the clustering diagnostics.

## Author

**Ajay Kumar Ginka Panasa** — Imperial College Business School AI/ML Programme.

🔗 [GitHub](https://github.com/GPAjay/Imperial_Executive_Education_Capstone2026)
