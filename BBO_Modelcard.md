# Model Card — BBO Strategy

## Overview

- **Name**: Hybrid Surrogate–Diagnostic BBO Strategy
- **Type**: Sequential black-box optimization with dual-surrogate ensemble (Gaussian Process + SVR), per-dimension diagnostic, and
  calibration-aware human-in-the-loop decision making.
- **Version**: Capstone implementation v1.

## Intended use

### Suitable for

- Sequential optimization of expensive black-box functions where each query is costly relative to the surrogate fit.
- Small-data, low-budget settings with tens of queries, low-to-moderate dimensionality (2D–8D).
- Methodological exercises in surrogate-driven optimization.
- Settings where human-in-the-loop judgment is permitted alongside the acquisition function.

### Use cases to avoid

- High-dimensional optimization (>10D) without methodological changes.
- Settings requiring formal global-optimality guarantees.
- For functions with high noise variance relative to signal, the calibration heuristic for this strategy breaks down.
- Statistical comparison against industrial-grade BBO methods (e.g. TuRBO, HEBO). Sample sizes here are far too small for such comparisons to be valid.
- Production deployment as an autonomous optimizer and the human-in-loop feedback step is structural to the strategy.

## Details — strategy evolution

The strategy evolved drastically across rounds. Three broad phases:

### Rounds 1–4 (Initial Exploration)

Coarse exploration using the initial dataset's GP fit. Submissions were driven primarily by GP-UCB/EI within a small search radius, with
little per-function specialization yet. Multiple functions remained flat or negative at this stage.

### Rounds 5–7 (Regime Detection)

Recognition that the functions occupied distinct regimes such as smooth-gradient, noisy, multimodal, bracketed-peak, accelerating,
volatile, steady-gain, broad-plateau. Realization that one strategy could not fit all. Per-function customization began: step sizes, candidate
patterns, and override logic were tailored to each function's observed behaviour.

### Rounds 8–11 (Model-data interplay)

Per-dimension nudge tests are systematically used to surface local gradients independent of the GP-EI recommendation. GP-EI was submitted
only when its own predicted mean exceeded the current best (the
"untrust rule"). Manual overrides became routine when the diagnostic
contradicted the principled candidate list. Reflection became explicit:
which assumptions had been validated by data, which had failed, and how
to calibrate boldness per function.

### Key techniques

1. **Gaussian Process regression** with kernel-and-alpha grid search
   (Matern ν=0.5/1.5/2.5, RBF; alpha 1e-8 to 1e-1) and per-dimension
   length-scale fitting.
2. **SVR regression** with C / gamma / epsilon grid search as a
   secondary surrogate to cross-check the GP.
3. **Per-dimension nudge test** — small symmetric perturbations from
   the current best to map local gradient direction and magnitude.
4. **GP-EI acquisition** with bounded search radius around the current
   best (typically ±0.05 to ±0.2).
5. **Manual candidate generation** reflecting the round's working
   hypothesis (e.g. "continue x5 gradient", "test untested dimension",
   "fine-tune known peak").
6. **Calibration-aware trust rule** — only follow the surrogate when
   its prediction at the current best is accurate to within a
   function-specific tolerance.

## Performance

### Metrics

- **Per-function final best y**, compared to the initial dataset's best.
- **GP calibration error at the current best** — distance between GP
  prediction and observed y, used as a trust signal.
- **Number of new-best wins per function** across the eleven rounds.
- **Gain-per-round trajectory** — used to distinguish accelerating
  regimes, steady regimes, and diminishing-returns regimes.

### Summary (qualitative)

- All eight functions showed at least one new best across R1–R11.
- Functions with smooth surrogate fits (calibration error < 1%) yielded
  emergent gains when bold steps were taken — including the project's
  largest single-round percentage gain.
- At least one function reached a clear noise floor by R10, with
  subsequent rounds confirming convergence rather than producing gains.
- Two functions showed monotonic surrogate-model improvement (rising
  marginal likelihood) while y gains decayed to noise — a case of model
  confidence and remaining headroom moving independently.
- One function exhibited persistent surrogate miscalibration (>5% error)
  across multiple rounds, where no probe strategy reliably improved on
  the running best.

### Failure modes observed

- Multi-dimensional GP-EI recommendations into combinations of known-bad
  coordinates (early-round losses).
- Per-dimension nudge predictions off by several percent at fine
  resolution on higher-dimensional functions.
- Surrogate models structurally unable to fit a function (SVR on the
  function with the widest y-range).

## Assumptions and limitations

### Assumptions

1. **Calibration error at the current best predicts trustworthiness
   elsewhere on the surface.** Verified on smooth functions; failed on
   at least one volatile function.
2. **Single-dimension moves are safer than multi-dimension moves on
   noisy or volatile functions.** Generally true, with a notable
   counter-example where a well-calibrated GP-EI multi-dim move paid
   off.
3. **The per-dimension nudge test surfaces all locally relevant
   gradients.** Worked on most functions, missed on at least one with
   sharp peaks at sub-step resolution.
4. **"Irrelevant dimension" classifications from the GP are real
   properties of the function.** May actually reflect insufficient
   variation in sampling, not a genuine flat axis.

### Limitations

1. Sample efficiency degrades sharply with dimensionality.
2. Sampling distribution is exploitation-biased — large fractions of
   the search space are entirely unsampled.
3. No formal global-optimality test is performed; the "function is at
   peak" conclusion is local-only.
4. The calibration-trust heuristic is applied uniformly across all
   eight functions, even though calibration behaviour varies
   substantially per function.
5. Statistical claims about strategy effectiveness are not possible
   from this sample size.

## Ethical considerations

### Transparency

- Every submission has an associated rationale recorded in the per-round
  JSON log and the strategy comment of the per-function script.
- The audit trail makes the strategy reproducible: a reviewer with the
  data, scripts, and reflections can reconstruct each submission and
  its reasoning chain.
- The weakest point in the audit trail is the human-override
  justification — sometimes documented only in candidate labels rather
  than in rigorous form.

### Reproducibility

- Grid searches, kernel choices, and candidate generations are
  deterministic given the dataset and random seed.
- Manual overrides require the contextual reasoning to be captured
  alongside the submission for full reproducibility — improving this
  documentation discipline would benefit any reader trying to repeat
  the work.

### Real-world adaptation

- The strategy generalizes to other small-data BBO settings, but the
  calibration-trust heuristic must be re-validated per function rather
  than transferred blindly.
- Boldness on one function does not predict boldness payoff on another;
  emergent gains are function-specific and cannot be assumed.
- The approach does NOT produce a generalizable model — each function's
  surrogate is local to its query distribution and not deployable
  outside the sampled neighbourhoods.

## Reflection on the model card itself

This card covers the necessary categories at a level appropriate for a
learning project. Additional detail intentionally omitted: per-function
numerical y trajectories (kept in the data logs), specific kernel-alpha
winners per round (kept in script outputs), and wall-clock runtimes
(not material to method performance). The current structure is
sufficient for the capstone's pedagogical purpose; expansion would
mainly help if the strategy were being packaged for external
deployment, which it is not.
