---
name: linear-program-objective-evaluation
description: Use when a pipeline parameter (e.g., a separation threshold Delta > 0)
  must be tuned by treating a downstream LP-solver/classifier accuracy as a black-box
  objective over a bounded search space.
when_to_use_negative:
- The downstream model's cell type classes are not linearly separable — the article
  notes this is a reasonable assumption in high dimensions but may not hold in lower
  dimensions
- The task requires nonlinear dimension reduction or manifold (trajectory/spatial)
  placement of unlabeled cells, which scGeneFit does not support
- The hyperparameter to tune is evaluated cheaply without any LP or classifier-in-the-loop
  step, making dual-annealing + LP backend overhead unnecessary
edam_topics:
- http://edamontology.org/topic_3307
- http://edamontology.org/topic_3474
tools:
- name: scGeneFit
  role: Marker-selection method whose constrained LP formulation supplies the LP solver
    used as the evaluation backend for candidate Delta values
  repo: github.com/solevillar/scGeneFit
- name: SciPy
  role: Provides scipy.optimize.dual_annealing, used to search the Delta space (Delta
    > 0) for the value maximizing LP-scored classifier accuracy
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-021-21453-4
    title: Optimal marker gene selection for cell type discrimination in single cell
      analyses
  source_content_hash: sha256:2dff9911cf76b7b5458e39aba64b14b5b8d8bea45f4a0b734d3b9dab0a1901db
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - SciPy
  - scGeneFit
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3307
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# linear-program-objective-evaluation

## Summary

Evaluates candidate hyperparameter values of an optimization-based marker-selection pipeline (e.g., the target separation Delta) by feeding them to a linear-program solver and scoring the resulting classifier's label-recovery accuracy, with scipy.optimize.dual_annealing locating the Delta that maximizes performance.

## When to use

Use when a pipeline parameter (e.g., a separation threshold Delta > 0) must be tuned by treating a downstream LP-solver/classifier accuracy as a black-box objective over a bounded search space.

## When NOT to use

- The downstream model's cell type classes are not linearly separable — the article notes this is a reasonable assumption in high dimensions but may not hold in lower dimensions
- The task requires nonlinear dimension reduction or manifold (trajectory/spatial) placement of unlabeled cells, which scGeneFit does not support
- The hyperparameter to tune is evaluated cheaply without any LP or classifier-in-the-loop step, making dual-annealing + LP backend overhead unnecessary

## Inputs

- Toy training set (scRNA-seq-like feature matrix with categorical cell labels)
- Test split for label-recovery accuracy scoring
- Classifier specification object
- LP solver module (Python) serving as the evaluation backend for candidate Delta values
- Delta search space bounds with positivity constraint Delta > 0

## Outputs

- hyperparameters file containing the best Delta (> 0) plus run metadata
- hyperparameter_optimizer.py — reusable dual-annealing optimization routine parameterized by arbitrary train/test/classifier combinations

## How to apply

Load the toy training set, test set, and classifier specification, then import the LP solver module (from the prior scGeneFit-constrained-optimization step) as the evaluation backend for candidate Delta values. Define the objective as label-recovery accuracy on the test split, computed by the LP solver for each Delta, and run scipy.optimize.dual_annealing over the search space subject to the positivity constraint Delta > 0 to locate the separation parameter maximizing classifier performance. Record the best Delta for the toy train/test split and serialize it with metadata into a hyperparameters file, saving the reusable dual-annealing routine as hyperparameter_optimizer.py so it can be invoked for arbitrary train/test/classifier combinations. This mirrors scGeneFit's rationale of formulating marker selection as a computationally tractable constrained optimization rather than a combinatorial gene-by-gene search.

## Related tools

- **scGeneFit** (Marker-selection method whose constrained LP formulation supplies the LP solver used as the evaluation backend for candidate Delta values) — github.com/solevillar/scGeneFit
- **SciPy** (Provides scipy.optimize.dual_annealing, used to search the Delta space (Delta > 0) for the value maximizing LP-scored classifier accuracy)

## Examples

```
from hyperparameter_optimizer import optimize_delta; best_delta = optimize_delta(X_train, y_train, X_test, y_test, lp_solver=task_005_solver)
```

## Evaluation signals

- hyperparameters file exists and contains a Delta entry strictly greater than 0
- hyperparameter_optimizer.py runs end-to-end on the toy split via the task_005 LP solver without errors
- Accuracy returned by the LP solver for the tuned Delta is >= accuracy at surrounding (suboptimal) Delta values on the toy test split
- Optimizer reproduces the recorded best Delta within tolerance when re-invoked on the same toy train/test/classifier combination
- Dual-annealing traces respect the positivity constraint (no evaluated Delta <= 0)

## Limitations

- The LP-scored objective inherits the linear separability assumption of scGeneFit, which may fail in lower dimensions
- Only categorical labeling is supported; no placement of unlabeled cells on trajectories or spatial manifolds
- Hyperparameter tuning is only meaningful when a working LP solver backend is available for the specific train/test/classifier combination

## Evidence

- [abstract] scGeneFit selects gene markers that jointly optimize cell label recovery using label-aware compressive classification methods.: "scGeneFit selects gene markers that jointly optimize cell label recovery using label-aware compressive classification methods."
- [intro] The marker choice task is combinatorially difficult, motivating the LP-based formulation and its hyperparameter search.: "The task of optimally choosing markers among all genes that most reliably and precisely distinguish cell labels given a hierarchical partitioning of cell type labels is a combinatorially difficult"
- [discussion] The linear-separability assumption underlying the LP-based method may not hold in lower dimensions, a caveat for tuned-Delta objectives.: "cell type classes are linearly separable, which is a reasonable assumption in high dimensions, but may not"
