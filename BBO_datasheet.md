# Datasheet — BBO Capstone Dataset

## Motivation

This dataset was created to support an iterative black-box optimization capstone project in which the maxima of eight unrelated functions must be found
within a strict query budget. Each round closes the loop between a small training set, a fitted surrogate model, and a single new submission, with the next
round's training set growing by exactly one point per function.

The dataset's purpose is twofold:

- A workbench for practising surrogate-driven sequential decision-making (Gaussian Process regression, SVR, expected-improvement acquisition).
- A record from which methodological reflection on scaling laws, emergent behaviour, sampling bias, and the limits of small-data optimization, can be drawn.

It is not intended as a general-purpose benchmark.

## Composition

The dataset consists of eight per-function tables (F1 through F8), each
containing the function's queried points and observed y values to date.

- **Functions**: F1 (Radiation Detection, 2D), F2 (Noisy BO, 2D), F3 (Drug Discovery, 3D), F4 (Warehouse Placement, 4D), F5 (Chemical Yield, 4D, Unimodal), F6 (Cake Recipe, 5D),
  F7 (Hyperparameter Tuning, 6D), F8 (8D Black-Box Optimisation).
- **Initial set sizes**: 10–40 points per function, supplied by the Capstone provider.
- **Per-round additions**: one query per function per round, appended weekly.
- **Row schema**: input vector X (length = function dimensionality), scalar response y, round-of-origin metadata.
- **Y ranges**: vary dramatically across functions. Some span many orders of magnitude into deeply negative regions; others remain in narrow positive bands.

The function labels are descriptive only, and the underlying generating functions are not disclosed by the capstone.

**Gaps**: queries cluster heavily around each function's running best.
Large fractions of each input space remain entirely unsampled. On at least one function, an exploration probe placed in an empty region returned a substantially better y than the cluster of refinement
queries had — direct evidence that the dataset is exploitation-biased and that uncovered regions may contain higher maxima.

## Collection process

Queries were generated weekly through a uniform per-function pipeline:

1. **Kernel-and-alpha grid search** to choose a tuned Gaussian Process.
2. **C-gamma-epsilon grid search** to choose a tuned SVR.
3. **Per-dimension nudge test** around the current best, to surface local gradients.
4. **GP-EI optimisation** within a bounded radius around the current best.
5. **Manual candidate list** reflecting the round's working hypothesis.
6. **Submission** selected from the combined candidate set, occasionally overriding both model recommendations and prior strategy when the
   diagnostic or human judgment justified it.

All candidate predictions, GP and SVR calibration errors, and the chosen submission are recorded in a global CSV tuning log.

## Preprocessing and uses

### Preprocessing

- Functions with extreme y dynamic ranges (orders of magnitude spread; positive and large-negative values) used a **symlog10 transform** before fitting the GP, with a `linthresh` chosen per function to preserve sensitivity near zero.
- Functions with bounded, modest y ranges used **raw y values**.
- SVR was fit on the same transformed y space where applicable.
- No outlier removal, smoothing, or imputation was applied; every queried point appears in the dataset.

### Intended uses

- A learning record for the capstone.
- The substrate for reflection questions on scaling laws and emergent behaviour.
- A benchmark for comparing decision strategies.

### Inappropriate uses

- Treating the recorded "best" as an estimate of the function's global maximum. Some maxima may live in regions never sampled.
- Using the surrogate fits to make predictions far outside the queried neighbourhoods.
- Comparing optimisation methods statistically — sample sizes are too small for rigorous claims.
- The dataset's dimension-relevance conclusions (e.g., "x2 is irrelevant on function F") should be read as local-to-sampled-region inferences, not global properties of the function.

## Distribution and maintenance

- Stored within this repository.
- Not published as a public benchmark — capstone-internal use under academic-honesty terms.
- Maintained by the learner for the duration of the capstone.
- The dataset is expected to grow before the capstone closes, after which it remains read-only for reflection and write-up purposes.
