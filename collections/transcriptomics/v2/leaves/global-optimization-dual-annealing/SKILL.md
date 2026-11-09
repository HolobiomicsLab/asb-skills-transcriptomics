---
name: global-optimization-dual-annealing
description: Use when a scalar hyperparameter (e.g. a target separation Delta) of
  an optimization-based method like scGeneFit must be tuned by maximizing a noisy,
  non-convex objective such as test-split label recovery accuracy, under a positivity
  constraint.
when_to_use_negative:
- The objective is convex or differentiable in closed form, where a local solver would
  suffice without global search
- Cell labels are not categorical (e.g. trajectories or spatial positions), since
  scGeneFit assumes categorical labeling and linear separability of cell type classes
edam_operation: null
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_3308
tools:
- name: SciPy
  role: Provides scipy.optimize.dual_annealing, the global optimizer run over the
    constrained Delta search space to maximize LP-evaluated classifier accuracy
- name: scGeneFit
  role: Marker selection method whose target separation Delta hyperparameter is being
    optimized; projects labeled samples into the low-dimensional marker space so same-label
    samples lie closer together
  repo: github.com/solevillar/scGeneFit
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-021-21453-4
    title: Optimal marker gene selection for cell type discrimination in single cell
      analyses
  source_content_hash: sha256:45f7351ae7e15b6acde3f027425db5e75810c6bbfcc8822aa50bdeeecdb51293
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - SciPy
  - scGeneFit
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# global-optimization-dual-annealing

## Summary

Uses scipy.optimize.dual_annealing to globally search a constrained scalar hyperparameter space (here, the target separation Delta > 0) by maximizing an objective such as classifier label-recovery accuracy evaluated through an external LP-solver backend. Encapsulates the search as a reusable optimizer module that serializes the best-found parameter with metadata.

## When to use

Use when a scalar hyperparameter (e.g. a target separation Delta) of an optimization-based method like scGeneFit must be tuned by maximizing a noisy, non-convex objective such as test-split label recovery accuracy, under a positivity constraint.

## When NOT to use

- The objective is convex or differentiable in closed form, where a local solver would suffice without global search
- Cell labels are not categorical (e.g. trajectories or spatial positions), since scGeneFit assumes categorical labeling and linear separability of cell type classes

## Inputs

- Toy training set (single-cell expression matrix with categorical cell labels)
- Held-out test split for evaluating label recovery accuracy
- Classifier specification (classifier used to score candidate Delta values)
- LP solver module (evaluation backend for candidate Delta values)

## Outputs

- hyperparameters file containing the best Delta entry (strictly > 0) with metadata
- hyperparameter_optimizer.py — reusable dual-annealing optimization routine for arbitrary train/test/classifier combinations

## How to apply

Load the toy training set, test split, and classifier specification, and import the LP-solver module that serves as the evaluation backend for candidate Delta values. Define the objective as classifier performance (label recovery accuracy on the test split) as a function of Delta, with the constraint Delta > 0, so that only separation parameters yielding strictly positive margins are admissible. Run scipy.optimize.dual_annealing over the Delta search space; dual annealing combines simulated annealing with local optimization, which suits the non-convex accuracy landscape produced by the label-aware compressive classification formulation of scGeneFit. Record the best Delta found for the toy train/test split and serialize it with metadata into a hyperparameters file. Wrap the routine as a reusable module (hyperparameter_optimizer.py) so it can be invoked for arbitrary train/test/classifier combinations.

## Related tools

- **SciPy** (Provides scipy.optimize.dual_annealing, the global optimizer run over the constrained Delta search space to maximize LP-evaluated classifier accuracy)
- **scGeneFit** (Marker selection method whose target separation Delta hyperparameter is being optimized; projects labeled samples into the low-dimensional marker space so same-label samples lie closer together) — github.com/solevillar/scGeneFit

## Examples

```
from hyperparameter_optimizer import optimize_delta; best = optimize_delta(X_train, y_train, X_test, y_test, classifier, lp_solver); # returns best Delta > 0 maximizing test label-recovery accuracy
```

## Evaluation signals

- hyperparameters file exists and contains a Delta entry strictly greater than 0
- hyperparameter_optimizer.py runs end-to-end on the toy split via the task_005 LP solver without errors
- The best Delta reported corresponds to maximal label recovery accuracy on the test split among evaluated candidates
- Optimizer reproduces the recorded best Delta when re-run on the same toy train/test split with the same classifier specification
- Optimizer accepts arbitrary train/test/classifier combinations (parameterized, not hard-coded to the toy split)

## Limitations

- Assumes linear separability of cell type classes, which is reasonable in high dimensions but may not hold in lower dimensions
- Only categorical labeling is supported; no support for placing unlabeled cells on trajectories or spatial manifolds
- Global search over a single scalar (Delta) is demonstrated; extension to higher-dimensional search spaces inherits dual-annealing runtime costs

## Evidence

- [abstract] scGeneFit selects gene markers that jointly optimize cell label recovery using label-aware compressive classification methods.: "scGeneFit selects gene markers that jointly optimize cell label recovery using label-aware compressive classification methods."
- [results] Method projects samples into a low-dimensional subspace so same-label samples are closer: "scGeneFit selected as markers the two features representing the circle coordinates whenever the size of the target marker set was ≥2"
- [discussion] Linear separability assumption limitation: "A key underlying assumption common to both scGeneFit and existing one-vs-all methods is that cell type classes are linearly separable"
- [discussion] Categorical labeling limitation / future manifold constraint: "we envision relaxing the categorical labeling to a manifold constraint that will allow selection of markers to place unlabeled cells at, for example, time points along cell trajectories"
