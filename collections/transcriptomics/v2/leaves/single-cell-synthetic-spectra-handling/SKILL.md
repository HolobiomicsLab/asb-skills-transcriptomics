---
name: single-cell-synthetic-spectra-handling
description: Use when a continuous positive separation/threshold parameter (Delta)
  must be tuned so that same-label samples stay closer than different-label samples
  in a reduced marker subspace, with accuracy on a held-out test split as the objective.
when_to_use_negative:
- The parameter to tune is discrete or combinatorial (e.g., choosing which genes constitute
  the marker set), where the article frames the selection as 'a combinatorially difficult'
  problem rather than a continuous annealing search
- Cell labels are continuous trajectories or spatial locations rather than categorical
  labels, since the method's underlying assumption is that 'cell type classes are
  linearly separable' under categorical labeling
- No LP solver or evaluation backend exists to score candidate Delta values, since
  the objective must be evaluated through the solver
tools:
- name: scGeneFit
  role: Marker-selection framework whose constrained optimization formulation defines
    the low-dimensional (selected-marker) subspace in which the separation parameter
    Delta is tuned
  repo: github.com/solevillar/scGeneFit
- name: SciPy
  role: Provides scipy.optimize.dual_annealing, used as the global optimizer over
    the Delta search space with the Delta > 0 positivity constraint
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-021-21453-4
    title: Optimal marker gene selection for cell type discrimination in single cell
      analyses
  source_content_hash: sha256:6bd0fe7d74f338d92d695dbfd4c1cb147d3e5449ccfec9994464646e3cbe5f16
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - SciPy
  - scGeneFit
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# single-cell-synthetic-spectra-handling

## Summary

A hyperparameter-search skill that reconstructs the target separation parameter (Delta) of a constrained marker-selection problem (scGeneFit-style label-aware compressive classification) by treating classifier accuracy, evaluated through a linear-program solver, as a black-box objective over Delta > 0 and optimizing it with scipy.optimize.dual_annealing.

## When to use

Use when a continuous positive separation/threshold parameter (Delta) must be tuned so that same-label samples stay closer than different-label samples in a reduced marker subspace, with accuracy on a held-out test split as the objective.

## When NOT to use

- The parameter to tune is discrete or combinatorial (e.g., choosing which genes constitute the marker set), where the article frames the selection as 'a combinatorially difficult' problem rather than a continuous annealing search
- Cell labels are continuous trajectories or spatial locations rather than categorical labels, since the method's underlying assumption is that 'cell type classes are linearly separable' under categorical labeling
- No LP solver or evaluation backend exists to score candidate Delta values, since the objective must be evaluated through the solver

## Inputs

- Toy training set (labeled samples)
- Test split with known labels for accuracy evaluation
- Classifier specification (classifier to be evaluated)
- LP solver module exposing the classification/evaluation backend
- Delta search-space bounds (strictly positive lower bound)

## Outputs

- hyperparameters file containing the best Delta (> 0) with metadata
- Reusable optimizer module (hyperparameter_optimizer.py) invocable for arbitrary train/test/classifier combinations

## How to apply

Load the toy training set, test set, and classifier specification supplied as inputs, and import the LP-solver module (e.g., emitted by a prior linear-programming task) as the evaluation backend. Define the objective as label-recovery accuracy on the test split as a function of the target separation Delta, so every candidate Delta is scored by routing the classification through the LP solver. Run scipy.optimize.dual_annealing over the Delta search space, encoding the positivity constraint Delta > 0 via a strictly positive lower bound (e.g., bounds starting just above 0). Serialize the best Delta with metadata into a hyperparameters file, and save the routine as a reusable Python module (hyperparameter_optimizer.py) that accepts arbitrary train/test/classifier combinations. Accept the result only if the hyperparameters file exists, contains a Delta strictly greater than 0, and the optimizer runs end-to-end on the toy split through the LP solver; the rationale is that scGeneFit formulates marker selection so samples with the same labels are closer together in the low-dimensional marker space than samples with different labels.

## Related tools

- **scGeneFit** (Marker-selection framework whose constrained optimization formulation defines the low-dimensional (selected-marker) subspace in which the separation parameter Delta is tuned) — github.com/solevillar/scGeneFit
- **SciPy** (Provides scipy.optimize.dual_annealing, used as the global optimizer over the Delta search space with the Delta > 0 positivity constraint)

## Examples

```
from scipy.optimize import dual_annealing; res = dual_annealing(lambda d: -lp_solver_accuracy(d, X_train, X_test, clf), bounds=[(1e-6, 10.0)]); best_Delta = res.x[0]
```

## Evaluation signals

- hyperparameters file exists and contains a Delta entry strictly greater than 0
- hyperparameter_optimizer.py runs end-to-end on the toy train/test split via the task_005 LP solver without errors
- The optimized Delta maximizes test-split label-recovery accuracy compared with other sampled Delta values from the annealing trajectory
- The optimizer module can be re-invoked on a different train/test/classifier combination and produce a valid positive Delta
- Same-label samples end up closer together than different-label samples in the Delta-induced low-dimensional marker subspace on the toy split

## Limitations

- A key underlying assumption is that cell type classes are linearly separable, which 'is a reasonable assumption in high dimensions, but may not' hold in lower dimensions
- No nonlinear dimension reduction methods are implemented; relaxing the linear separability assumption is only planned future work
- Marker selection is limited to categorical labeling; placing unlabeled cells on trajectories or spatial assays (manifold constraint) is envisioned but not supported

## Evidence

- [abstract] scGeneFit selects gene markers that jointly optimize cell label recovery using label-aware compressive classification methods.: "scGeneFit selects gene markers that jointly optimize cell label recovery using label-aware compressive classification methods."
- [intro] Combinatorial difficulty of marker choice: "The task of optimally choosing markers among all genes that most reliably and precisely distinguish cell labels given a hierarchical partitioning of cell type labels is a combinatorially difficult"
- [results] Toy-model marker recovery: "scGeneFit selected as markers the two features representing the circle coordinates whenever the size of the target marker set was ≥2"
- [discussion] Linear separability assumption: "A key underlying assumption common to both scGeneFit and existing one-vs-all methods is that cell type classes are linearly separable, which is a reasonable assumption in high dimensions, but may not"
