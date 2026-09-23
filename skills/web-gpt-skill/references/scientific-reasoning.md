# Scientific reasoning mode

Use for climate, hydrology, environmental science, soil freeze-thaw, land-surface modelling, observations, statistics, or other research tasks.

## Required distinctions

Ask Web GPT to label or clearly separate:
- **Observation:** directly measured/retrieved evidence.
- **Model output:** simulated quantity, not observation.
- **Assumption:** imposed premise or simplification.
- **Hypothesis:** testable explanation.
- **Inference:** conclusion supported by evidence plus assumptions.
- **Causal claim:** requires stronger support than association.

## Process vs metric

When a process change appears physically better but aggregate metrics degrade, explicitly test whether the disagreement comes from:
- baseline/state bias
- anomaly/dynamic skill
- phase/amplitude error
- event timing error
- hysteresis/path dependence
- threshold/selection effects
- depth, footprint, temporal-resolution, or representativeness mismatch
- compensating errors in the control model

Do not equate lower RMSE with mechanistic correctness.

Prefer diagnostics that separate state and process, for example when applicable:
- bias vs ubRMSE/anomaly error
- event-conditioned response
- slopes/sensitivities such as ΔY/ΔX
- warming vs cooling branches
- phase lag and event timing
- conditional distributions
- paired site/depth comparisons
- physically constrained residuals

## Competing explanations

Require at least the strongest plausible alternative explanation and one discriminating diagnostic for each major hypothesis.

## Publication interpretation

When asked to write interpretation, Web GPT should distinguish:
1. what the data show;
2. what mechanism is consistent with the data;
3. what is not yet demonstrated;
4. which diagnostic would strengthen a causal/process claim.
