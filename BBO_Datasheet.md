# Datasheet — BBO Capstone Dataset

A datasheet documenting the dataset assembled over the black-box optimisation capstone project.

## Motivation

This dataset was created to support an iterative black-box optimisation capstone in which the maxima of eight unrelated functions must be found within a strict query budget of 13 weeks (one query per function per round). Each round iteration: a small training set is used to fit a surrogate model, the model informs a single new submission, and the returned value grows the training set by exactly one point per function.

The dataset serves two purposes:

- A workbench for practising surrogate-driven sequential decision-making (Gaussian Process regression, SVR cross-checking, expected-improvement acquisition, and clustering diagnostics).
- A record from which methodological reflection can be drawn on sampling bias, model calibration, convergence, and the limits of small-data optimisation.

It is not intended as a general-purpose benchmark.

## Composition

The dataset consists of eight function tables (F1 through F8), each containing the queried input points and observed outputs accumulated
over the capstone challenge.

- **Functions and dimensionality**: F1 (Radiation Detection, 2D),
  F2 (Noisy Model, 2D), F3 (Drug Discovery, 3D),
  F4 (Warehouse Placement, 4D), F5 (Chemical Yield, 4D, unimodal),
  F6 (Cake Recipe, 5D), F7 (Hyperparameter Tuning, 6D),
  F8 (8D Optimisation, 8D).
- **Initial set sizes**: 10 to 40 seed observations per function
  (varying by function), supplied by the capstone provider as `.npy` files.
- **Per-round additions**: one query per function per round, appended weekly across 13 rounds.
- **Row schema**: input vector X (length = function dimensionality, all  values in [0, 1]), scalar response y, and round-of-origin metadata.
- **Output ranges**: vary dramatically across functions. Some span several orders of magnitude and reach deeply negative values; others
  stay within narrow positive bands. This heterogeneity drove  per-function preprocessing choices (see below).

The function labels are descriptive scenarios only; the underlying generating functions are never disclosed. Only inputs and outputs are
observable.

**Known gaps and biases.** Queries cluster heavily around each
function's running best, so large fractions of every input space remain
entirely unsampled. On at least one function, a deliberate exploration
probe placed in an empty region returned a substantially better output
than many rounds of local refinement near the previous best had — direct
evidence that the sampling distribution is exploitation-biased and that
uncovered regions may contain higher maxima.

## Collection process

Each round's query was generated through a uniform per-function pipeline:

1. **GP tuning** — kernel-and-alpha grid search (Matérn ν = 0.5 / 1.5 /
   2.5 and RBF; alpha from 1e-8 to 1e-1) selecting the fit with the
   highest log marginal likelihood, with per-dimension length scales.
2. **SVR tuning** — C / gamma / epsilon grid search, fitting a second
   surrogate to cross-check the GP.
3. **Per-dimension nudge test:** Small symmetric perturbations around the current best in each dimension, mapping local gradient direction
   and magnitude.
4. **EI/ UCB optimisation:** Expected-improvement and upper confidence bound search within a bounded radius around the current best.
5. **Manual candidate list:** Specified points encoding the round's working hypothesis (continue a gradient, test an untested
   dimension, fine-tune a known peak).
6. **Submission:** Chosen from the combined candidate set, sometimes overriding the model recommendation when the diagnostic or explicit
   reasoning justified it.

All candidate predictions, GP and SVR calibration errors at the current best, and the chosen submission were logged per round (per-function JSON logs plus a global CSV tuning log). The project ran over 13 calendar
weeks.

## Preprocessing and uses

### Preprocessing

- Functions with extreme output ranges (orders-of-magnitude spread,
  including large negatives) were fitted on a **symlog10 transform**,
  with a `linthresh` chosen per function to preserve sensitivity near
  zero.
- Functions with bounded, modest output ranges were fitted on **raw y
  values**.
- SVR was fitted on the same transformed space as the GP where
  applicable.
- No outlier removal, smoothing, or imputation was applied to the stored
  dataset. Every queried point is retained; transforms are applied at
  fit time, not baked into the data.

### Intended uses

- A learning record for the capstone.
- The substrate for the weekly methodological reflections.
- A basis for comparing decision strategies (model-driven vs. human-override, single-dimension vs. multi-dimension moves, exploration vs. exploitation).

### Inappropriate uses

- Treating the recorded best as the function's global maximum — some maxima may lie in regions never sampled.
- Using the surrogate fits to predict far outside the queried neighbourhoods.
- Comparing optimisation methods statistically — sample sizes are far  too small for rigorous claims.
- Reading "irrelevant dimension" conclusions from the GP as global properties. They are local-to-sampled-region inferences and may
  reflect insufficient sampling variation rather than a genuinely flat axis.

## Distribution and maintenance

- Stored within this repository, organised by function.
- Not published as a public benchmark — capstone use under academic terms.
- Maintained by the project owner for the duration of the capstone.
- After the final round, the dataset is read-only and retained for reflection and write-up.
