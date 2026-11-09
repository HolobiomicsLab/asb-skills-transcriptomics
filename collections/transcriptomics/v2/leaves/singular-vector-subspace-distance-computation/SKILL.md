---
name: singular-vector-subspace-distance-computation
description: Use when benchmarking co-expression estimators on simulated data with
  known ground-truth co-expression structure, and you need a metric for recovery of
  the dominant principal subspace (e.g., top four singular vectors) beyond value-level
  correlations.
when_to_use_negative:
- Input is raw UMI count data with no estimated or ground-truth co-expression matrices
  — the metric requires both matrices, not counts
- Comparing clustering quality only (use adjusted Rand index of co-expression cluster
  or expression-cluster recovery instead)
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: CSCORE (R package)
  role: Produces the estimated co-expression networks on simulated counts whose leading
    singular vectors are compared against the latent truth via subspace distance
  repo: https://github.com/ChangSuBiostats/CS-CORE
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-023-40503-7
    title: Cell-type-specific co-expression inference from single cell RNA-sequencing
      data
  source_content_hash: sha256:60731321c1ba187a7f66964c1f8a6079e201060bbc049391b5c4af5746d8af3f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ChangSuBiostats/CS-CORE
  tools:
  - CS-CORE
  - CSCORE (R package)
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# singular-vector-subspace-distance-computation

## Summary

Computes the subspace distance between the top singular vectors of an estimated co-expression matrix and the latent true co-expression matrix, as used in the CS-CORE Fig. 4a-e simulation benchmark to quantify how faithfully a method recovers the principal subspace driving co-expressed gene clusters.

## When to use

Use when benchmarking co-expression estimators on simulated data with known ground-truth co-expression structure, and you need a metric for recovery of the dominant principal subspace (e.g., top four singular vectors) beyond value-level correlations.

## When NOT to use

- Input is raw UMI count data with no estimated or ground-truth co-expression matrices — the metric requires both matrices, not counts
- Comparing clustering quality only (use adjusted Rand index of co-expression cluster or expression-cluster recovery instead)

## Inputs

- Estimated co-expression matrix (p x p) from a method applied to simulated UMI counts (n=2000 cells, p=100 genes)
- Latent true co-expression matrix (p x p) from the simulated latent-variable model with four co-expression clusters

## Outputs

- Per-method subspace distance between the top four singular vectors of estimated and true co-expression matrices
- Row of the figure4_simulation_benchmark.csv results table matching the Fig. 4a-e panels

## How to apply

After generating UMI counts via the Poisson measurement model (varying sequencing depths and measurement error across cells) over p=100 genes forming four co-expression clusters, estimate the co-expression matrix with each method (CS-CORE via the CSCORE R package, plus comparison methods). Take the top four singular vectors from both the estimated and the latent true co-expression matrices via SVD, project each onto its span, and compute the subspace distance between the two leading 4-dimensional subspaces. A small subspace distance indicates the method preserved the principal directions that define the four co-expressed clusters, complementing value-level metrics such as correlation of estimated vs true co-expression values. Assemble this per-method metric alongside adjusted Rand index of cluster recovery and correlation of estimated vs true means into a results table matching Fig. 4a-e (saved as figure4_simulation_benchmark.csv).

## Related tools

- **CSCORE (R package)** (Produces the estimated co-expression networks on simulated counts whose leading singular vectors are compared against the latent truth via subspace distance) — https://github.com/ChangSuBiostats/CS-CORE

## Examples

```
Vs <- svd(true_coexp)$v[, 1:4]; Ve <- svd(est_coexp)$v[, 1:4]; dist <- norm(Vs %*% t(Vs) - Ve %*% t(Ve), "F")
```

## Evaluation signals

- Subspace distance for CS-CORE should be markedly smaller than for comparison methods, since only CS-CORE accurately recovered the truly co-expressed gene clusters in the Fig. 4 simulations
- Distance is computed consistently using the same number (top four) of singular vectors from both estimated and true matrices
- Results table reproduces the structure of Fig. 4a-e panels and is saved as figure4_simulation_benchmark.csv
- Per-method subspace distance should be consistent with the same method's adjusted Rand index of co-expression cluster recovery (methods with high ARI should show low subspace distance)

## Limitations

- Most existing methods suffered from inflated false positives and biased co-expression estimates, so their subspace distances can be strikingly inaccurate even when count-level statistics appear reasonable
- The metric depends on the ground-truth co-expression matrix being available from simulation; it cannot be computed on real datasets without a latent reference

## Evidence

- [other] subspace distance between top four singular vectors of estimated and true co-expression matrices: [evidence span withheld — anchor status: not_found]
- [abstract] biased estimates and clustering: "most existing methods suffered from inflated false positives as well as biased co-expression estimates and clustering analysis"
- [abstract] CS-CORE accuracy: "whereas CS-CORE gave accurate estimates in these experiments"
