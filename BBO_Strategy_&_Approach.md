## 🔬 Strategy & Approach

### Core Method

- **Surrogate Model:** Gaussian Process (GP) Regression with Matérn kernel (ν=2.5)
- **Acquisition Functions:** Upper Confidence Bound (UCB) + Expected Improvement (EI) — ensemble
- **Beta Tuning:** Adaptive per-function beta values to balance exploration vs exploitation
- **Trust Regions:** Constrain search to neighbourhood of confirmed best known point (Week 4+)
- **Candidate Sampling:** 50,000 Latin Hypercube points per function

### 🔄 How the Pipeline Works

1. **Load data** — stack initial `.npy` observations with all weekly submissions and results
2. **Log-transform outputs** — stabilise GP fitting across extreme output ranges (e.g. F5 at 3651 vs F1 at 0.097)
3. **Build candidate grid** — 50,000 LHS points, constrained to trust region around confirmed best X
4. **Fit GP** — Matérn kernel (ν=2.5), `normalize_y=True`, 3 restarts
5. **Score candidates** — compute UCB and EI across all candidates
6. **Ensemble selection** — pick candidate with higher GP posterior mean between UCB and EI winners
7. **Duplicate check** — auto-perturb if selected point is within 0.015 of any prior submission
8. **Manual override** — when GP contradicts consistent empirical trend, trust the trend
9. **Output** — portal submission string in `x1-x2-...-xn` format (6 decimal places)


### Key Techniques

- Log transforms for functions with extreme output ranges
- Yeo-Johnson transform for volatile functions (F4)
- 50,000 LHS candidate points per function for broad coverage
- Trust regions anchored to confirmed all-time best X (not GP best)
- Boundary clipping to [0.02, 0.98] — corner points consistently perform poorly
- Duplicate check to prevent resubmitting previously queried points
- Outlier removal before GP fit (F4: W2/W3 values of −26.59 and −26.07 excluded permanently)
- Recent-only GP for non-stationary functions (F4: uses only post-W4 data)
- Per-function policies: MOMENTUM / RECOVERY / NEAR-EXACT / RIDGE / BOLD-EXPLORE



### Final Per-Function Results Summary

| Function | All-time Best | Best Week | Final Strategy That Worked |
|----------|-------------|-----------|--------------------------|
| F1 | **0.33641** ⭐ | **W13** | x2=0.654 locked; x1=0.635 (not 0.6515!) found true peak |
| F2 | 0.6478 | W6 | Stochastic — best draw at (0.7049, 0.9214) was W6 |
| F3 | -0.0107 | W1 | x2=0.9699 + x3=0.4748 + x1≈0.02 — W1 result never beaten |
| F4 | -0.1284 | W4 | Non-stationary; W4 region unreproducible — W7 region safer |
| F5 | **4039.88** ⭐ | **W13** | Remove 0.98 cap on x2-x4; x1=0.027 ridge peak |
| F6 | -0.2037 | W8 | x4=0.718, x5=0.020 critical; W8 result never beaten |
| F7 | **3.1050** ⭐ | **W13** | Score-weighted centroid of W10/W11/W12 confirmed bests |
| F8 | 9.9799 | W12 | Trend extrapolation W10→W11→W12; 4 consecutive new bests |

---


### 📝 Lessons Learned
| Week | Lesson |
|------|--------|
| W1 | Broad exploration works well early — 6/8 improved from initial GP+UCB scan |
| W2 | Aggressive UCB on noisy functions causes extreme outliers (F4: −26.59) — always exclude before fitting |
| W3 | High beta + SVM region filter on < 15 points = overexploration disaster. Never use SVM as region filter |
| W4 | Trust regions are essential after regression; EI is more conservative than UCB alone |
| W5 | Anchor trust center to all-time best X, not GP best. Dual-strategy comparison useful for stuck functions |
| W6 | Per-function beta tuning outperforms a single global beta — each function needs different explore/exploit balance |
| W7 | Drifting even 0.05 in x2 for F1 crashed output from 0.088 → 3.88e-13. Stick to confirmed best coordinates |
| W8 | Full LHS for stuck functions produced worst-ever F3 result. Better to anchor to historical best than explore blindly |
| W9 | Verify GP suggestion direction against empirical trends — GP suggested decreasing x2 for F5; data said keep it at 0.979 |
| W10 | F4 shows non-stationarity: same W4 coords gave −0.128 in W4 and −14.5 in W9. Use recent-only GP for non-stationary landscapes |
| W11 | Manual overrides beat raw GP when empirical evidence is clear — F5 ridge peak at x1=0.027 outperformed GP's preference to push lower |
| W12 | Ridge width: x3 deviation of just 0.016 (from 0.9795 to 0.963) caused 188-point drop in F5. Both x3 and x4 must stay at 0.979-0.980 on the ridge. |
| W12 | x2 in F1 is an extremely narrow spike: x2=0.670 (Δ=+0.016 from optimal 0.654) crashed output from 0.097 to −0.003. Never deviate more than 0.001 in x2 for F1. |
| W13 | Boundary caps can cost more than expected: removing the [0.02, 0.98] clip on F5's x2-x4 gained +388 points (+10.6%) in a single query. Always test at-boundary when a ridge is trending toward the edge. |
| W13 | The true peak of F1 (0.33641) was never near 0.6515 — it was at x1=0.635. A 0.016 shift in x1 (not x2) tripled the all-time best. Even well-explored functions can have untested neighbourhoods. |
| W13 | Score-weighted centroids (softmax weighting over confirmed-best observations) are a simple, powerful way to handle non-stationarity: instead of trusting one best point, weight all good points and let the centroid naturally drift toward the most reliable region. |

---

## 🛠️ Technologies Used
| Category | Technologies |
|----------|-------------|
| Language | Python |
| Libraries | NumPy, Matplotlib, Scikit-Learn, SciPy |
| Models | Gaussian Process (Matérn ν=2.5) |
| Sampling | Latin Hypercube Sampling (scipy.stats.qmc) |
| Transforms | Log, Yeo-Johnson (for volatile functions like F4) |
| Tools | Jupyter Notebooks, Git, GitHub |

---
