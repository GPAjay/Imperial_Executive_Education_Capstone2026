# Model Card: BBO Challenge

A model card documenting the *optimisation approach* used in this capstone. Follows the Modelcard framework described in the course: overview, intended use, details, performance, assumptions and limitations, ethical considerations.

## Overview

- **Name**: Hybrid Surrogate–Diagnostic BBO Strategy
- **Type**: Sequential black-box optimisation combining a dual-surrogate ensemble (Gaussian Process + SVR), a per-dimension gradient diagnostic, and calibration-aware human-in-the-loop decision-making.
- **Version**: Capstone implementation, 13 rounds (v1).
- **Objective**: Maximise the output of each of eight unknown functions within a fixed query budget.

## Intended use

### Suitable for

- Sequential optimisation of expensive black-box functions, where each evaluation is costly relative to the surrogate fit.
- Small-data, low-budget settings — tens of queries, low-to-moderate dimensionality (2D–8D).
- Situations where human judgment is permitted alongside an acquisition function.
- Methodological learning and teaching of surrogate-driven optimisation.

### Use cases to avoid

- High-dimensional optimisation (>10D) without methodological changes.
- Settings requiring formal global-optimality guarantees.
- Functions with high noise variance relative to signal — the calibration-trust heuristic that anchors this strategy will degrade.
- Statistical benchmarking against industrial BBO methods (e.g. TuRBO, HEBO); the sample sizes here are far too small for valid comparison.
- Fully autonomous production deployment — the human-in-the-loop step is structural to the strategy.

## Details — strategy evolution

The strategy evolved substantially over the course of the project, classified into three broad phases.

### Phase 1 — Exploration (early rounds)

Coarse exploration from the seed data, driven mainly by Expected-Improvement (EI) and Upper Confidence Bound (UCB)  search within a small radius, with little function-based specialisation. Several functions stayed flat or negative at this stage.

### Phase 2 — Regime detection (middle rounds)

Recognition that the eight functions occupied several distinct regimes, such as smooth-gradient, noisy, multimodal, bracketed-peak, accelerating, volatile, steady-gain, and broad-plateau. No single strategy fits all of the regimes. Per-function customisation commences: step sizes, candidate patterns, and override logic are tailored to each function's observed behaviour.

### Phase 3 — Model–data interplay and clustering diagnostics (late rounds)

Per-dimension nudge tests were used systematically to surface local gradients independent of the acquisition function. Expected-improvement
candidates were submitted only when the GP's own predicted mean at the candidate exceeded the current best (the "untrust rule").

K-means clustering was introduced as a diagnostic to characterise the search space per function, separating high-value regions from exploration scatter, confirming multimodality where present, and revealing which dimensions have never been varied inside the best region.
Manual overrides are done where the evidence is overwhelming.

### Key techniques

1. **Gaussian Process regression:** kernel-and-alpha grid search (Matérn ν = 0.5 / 1.5 / 2.5 and RBF; alpha 1e-8 to 1e-1), selecting
   by log marginal likelihood, with per-dimension length scales.
2. **SVR regression:** C / gamma / epsilon grid search as a secondary surrogate to cross-check the GP (and to detect when a surrogate is structurally unable to fit a function).
3. **Per-dimension nudge test:** Small symmetric perturbations from the current best, mapping local gradient direction and magnitude.
4. **EI acquisition:** Bounded-radius expected-improvement search (typically ±0.05 to ±0.2 around the current best).
5. **Manual candidate generation** points encoding the round's working hypothesis.
6. **Calibration-trust rule:** Follow the surrogate only when its prediction at the current best is accurate to within a
   function-specific tolerance; scale boldness inversely with that error.
7. **K-means clustering diagnostics:** Function-wise cluster characterisation (with silhouette selection and, for volatile
   functions, an auxiliary output-volatility metric) to distinguish converged peaks from noise and to expose sampling gaps.

## Performance

### Metrics

- **Function's best output**.
- **GP calibration error at the current best** — the gap between GP prediction and observed value, used directly as a trust signal.
- **New-best (% gain) per function** across the BBO challenge.
- **Gain-per-round trajectory** used to distinguish accelerating, steady, and diminishing-returns regimes.

### Summary of outcomes

- Every function improved on its seed-data best at least once during the
  campaign; the final round alone produced four new bests.
- On smooth, well-calibrated functions (calibration error well under
  1%), bold multi-dimensional moves produced the largest emergent gains
  — including a single-round improvement of roughly +85% on one function
  by testing an unexplored boundary, and a bold acquisition-driven move
  on another that gained ~+19% mid-campaign and a further ~+39% when the
  same validated pattern was re-applied in the final round.
- Several functions reached a clear noise floor mid-to-late campaign;
  subsequent rounds confirmed convergence rather than producing gains.
- On the most volatile function, a restrained minimal-step approach
  finally beat a long-standing best in the last round — a reminder that
  patience can outperform aggressive probing on noisy landscapes.
- At least one function showed monotonically rising model confidence
  (log marginal likelihood climbing round over round) while actual
  gains decayed to noise — a clean demonstration that model confidence
  and remaining headroom are independent.
- One function exhibited persistent surrogate miscalibration (GP error
  well above 5%) throughout; there, no probe strategy reliably beat the
  running best, and the correct response was to minimise step size and
  distrust the model.

### Failure modes observed

- Multi-dimensional acquisition recommendations that extrapolated into
  combinations of known-bad coordinates (early-round losses, and a
  notable mid-campaign loss when a candidate crossed into the valley
  between two modes of a bimodal function).
- Per-dimension nudge predictions off by several percent at fine
  resolution on the higher-dimensional functions.
- A surrogate structurally unable to fit a function — SVR consistently
  hallucinated on the widest-output-range functions and had to be
  ignored there.

## Assumptions and limitations

### Assumptions

1. **Calibration error at the current best predicts trustworthiness elsewhere.** Held on smooth functions; failed on at least one volatile function.
2. **Single-dimension moves are safer than multi-dimension moves on noisy/volatile functions.** Generally true with a notable
   counter-example where a well-calibrated multi-dimensional move paid off greatly.
3. **The per-dimension nudge test surfaces all locally relevant gradients.** Worked on most functions; missed a sharp peak that lived below the nudge resolution on one function.
4. **The GP's "irrelevant dimension" classifications are real.** They may instead reflect insufficient sampling variation, not a genuinely
   flat axis.

### Limitations

1. Sample efficiency degrades sharply with dimensionality.
2. The sampling distribution is exploitation-biased; large regions stay unsampled.
3. No formal global-optimality test is performed — Maxima conclusions are local only.
4. The calibration heuristic is applied uniformly, even though calibration behaviour varies markedly per function.
5. Sample sizes are too small for statistical claims about strategy effectiveness.

## Ethical considerations

### Transparency

- Every submission has a recorded rationale and a strategy note in the code, so a reviewer with the data, scripts, and reflections can
  reconstruct the decision.
- The weakest point in the audit trail is the human-override justification, which is not captured with full rigour in every case.

### Reproducibility

- Grid searches, kernel choices, and candidate generation are deterministic given the dataset and random seed.
- Human-in-the-loop decisions depend on contextual reasoning that must be recorded alongside the submission for full reproducibility.

### Real-world adaptation

- The approach generalises to other small-data BBO settings, but the calibration heuristic must be re-validated per function, and cannot be
  transferred directly.
- Payoffs are function-specific: an exploratory move that succeeded on one function is not evidence that it will succeed on another.
- The approach does not produce a reusable predictive model: each surrogate is local to its query distribution and is not deployable
  outside the sampled neighbourhoods.

## Reflection on this model card

This card covers the categories at a depth appropriate for a learning project. Functions' numerical trajectories, per-round kernel/alpha
winners, and wall-clock runtimes are intentionally omitted as they would add length without changing the methodological picture. For a production system, the human-override reasoning would need to be logged more formally, that is the single change that would most improve the card's usefulness.
