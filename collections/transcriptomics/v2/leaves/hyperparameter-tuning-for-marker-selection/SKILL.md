---
name: hyperparameter-tuning-for-marker-selection
description: Use when a marker-selection pipeline (e.g. scGeneFit) exposes a tunable
  separation parameter Delta that controls how closely same-label samples must sit
  in the selected-marker subspace, and you must locate the Delta maximizing label-recovery
  accuracy.
when_to_use_negative:
- The marker-selection method has no tunable separation parameter and marker set size
  alone fully specifies the search (e.g. scGeneFit's spectral toy model where any
  target marker set size ≥ 2 recovers circle-coordinate labels deterministically).
- Cell-type classes are not linearly separable in the low-dimensional selected-marker
  space — scGeneFit assumes linear separability, so an LP-evaluated Delta objective
  would be mis-specified.
- Labels are continuous or manifold-valued (e.g. trajectory time points or spatial
  locations); the approach presumes categorical cell labels.
edam_operation: http://edamontology.org/operation_2425
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: scGeneFit
  role: Marker-selection method whose label-aware compressive classification objective
    the tuned Delta parameter feeds; projects labeled samples into the selected-marker
    subspace so same-label samples are closer than different-label ones
  repo: github.com/solevillar/scGeneFit
- name: SciPy
  role: Provides scipy.optimize.dual_annealing, the global optimizer run over the
    Delta search space under the Delta > 0 constraint
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-021-21453-4
    title: Optimal marker gene selection for cell type discrimination in single cell
      analyses
  source_content_hash: sha256:073e44cd6fd20508b2850ec7be6d32b57e9094e4f352b597dff93705cddb5aed
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - SciPy
  - scGeneFit
  edam_operation: http://edamontology.org/operation_2425
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# hyperparameter-tuning-for-marker-selection

## Summary

Reconstructs a dual-annealing hyperparameter search over the target separation Delta for marker selection, scoring candidate Delta values by classifier label-recovery accuracy computed through a linear-program solver backend. It serializes the best Delta with metadata so scGeneFit-style marker selection can be reused on arbitrary train/test/classifier splits.

## When to use

Use when a marker-selection pipeline (e.g. scGeneFit) exposes a tunable separation parameter Delta that controls how closely same-label samples must sit in the selected-marker subspace, and you must locate the Delta maximizing label-recovery accuracy.

## When NOT to use

- The marker-selection method has no tunable separation parameter and marker set size alone fully specifies the search (e.g. scGeneFit's spectral toy model where any target marker set size ≥ 2 recovers circle-coordinate labels deterministically).
- Cell-type classes are not linearly separable in the low-dimensional selected-marker space — scGeneFit assumes linear separability, so an LP-evaluated Delta objective would be mis-specified.
- Labels are continuous or manifold-valued (e.g. trajectory time points or spatial locations); the approach presumes categorical cell labels.

## Inputs

- Toy training set (samples with categorical cell-type labels)
- Test set with labels for accuracy evaluation
- Classifier specification
- LP solver module (evaluation backend for candidate Delta values)
- Delta search space definition with positivity constraint Delta > 0

## Outputs

- hyperparameters artifact containing the best Delta (strictly > 0) with metadata
- hyperparameter_optimizer.py — reusable dual-annealing optimization routine for arbitrary train/test/classifier combinations

## How to apply

Load the toy training set, test set, and classifier specification as inputs, and import the LP solver module (produced by the preceding linear-program step) as the evaluation backend. Define the objective as label-recovery accuracy on the test split as a function of the target separation Delta, with the positivity constraint Delta > 0. Run scipy.optimize.dual_annealing over the Delta search space to maximize classifier performance, then record the best Delta for the toy train/test split. Serialize the chosen Delta with metadata into a hyperparameters artifact and save the optimization routine as a reusable Python module (hyperparameter_optimizer.py) that can be invoked for arbitrary train/test/classifier combinations. Rationale: scGeneFit jointly optimizes cell label recovery via label-aware compressive classification, so Delta must be fit against measured label recovery rather than fixed arbitrarily.

## Related tools

- **scGeneFit** (Marker-selection method whose label-aware compressive classification objective the tuned Delta parameter feeds; projects labeled samples into the selected-marker subspace so same-label samples are closer than different-label ones) — github.com/solevillar/scGeneFit
- **SciPy** (Provides scipy.optimize.dual_annealing, the global optimizer run over the Delta search space under the Delta > 0 constraint)

## Examples

```
from hyperparameter_optimizer import optimize_delta; best_delta = optimize_delta(X_train, y_train, X_test, y_test, lp_solver, bounds=[(1e-6, None)], method='dual_annealing')
```

## Evaluation signals

- hyperparameters artifact exists and contains a Delta entry strictly greater than 0
- hyperparameter_optimizer.py runs end-to-end on the toy train/test split via the task_005 LP solver without errors
- The dual-annealing objective (test-split label-recovery accuracy at the found Delta) is at least as high as at surrounding candidate Delta values in the search space
- Rerunning the optimizer on the same toy split with a fixed seed reproduces a comparable best-Delta and accuracy
- The serialized hyperparameters include the train/test split and classifier specification as metadata

## Limitations

- scGeneFit and this tuning loop assume cell-type classes are linearly separable, which is reasonable in high dimensions but may not hold in lower-dimensional selected-marker spaces.
- No nonlinear dimension-reduction methods are supported, so the Delta objective is constrained to linear label-aware projections.
- Marker selection is limited to categorical labeling — it does not support placing unlabeled cells on trajectories or in spatial assays; relaxing this to a manifold constraint is envisioned future work.

## Evidence

- [abstract] scGeneFit selects gene markers that jointly optimize cell label recovery using label-aware compressive classification methods.: "scGeneFit selects gene markers that jointly optimize cell label recovery using label-aware compressive classification methods."
- [results] scGeneFit selected as markers the two features representing the circle coordinates whenever the size of the target marker set was ≥2: "scGeneFit selected as markers the two features representing the circle coordinates whenever the size of the target marker set was ≥2"
- [discussion] A key underlying assumption common to both scGeneFit and existing one-vs-all methods is that cell type classes are linearly separable: "A key underlying assumption common to both scGeneFit and existing one-vs-all methods is that cell type classes are linearly separable"
- [intro] we developed scGeneFit, a rigorous and efficient approach for marker selection in the context of scRNA-seq data with a given hierarchical partition of labels: "we developed scGeneFit, a rigorous and efficient approach for marker selection in the context of scRNA-seq data with a given hierarchical partition of labels"
