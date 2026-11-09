---
name: marker-selection-stability-analysis
description: Use when you need to compare the run-to-run stability of marker sets
  from phenotype-cover solvers (GreedyPC/CEMPC) against global methods (ReliefF, F
  values, mRMR) on multi-class scRNA-seq phenotype discrimination tasks.
when_to_use_negative:
- When the goal is discriminative power or classification accuracy of the marker set
  rather than reproducibility of the selected features — use the paper's train-on-subset/logistic-regression
  evaluation workflow instead
- When only a single deterministic selection is needed and sampling variance is irrelevant;
  stability testing requires repeated runs over random half-samples (5 runs of 50%
  of the data)
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: phenotype-cover
  role: Provides the GreedyPC and CEMPC solvers whose selected marker sets are compared
    across resampling runs; installed via pip install phenotype-cover (and multiset-multicover)
  repo: https://github.com/euxhenh/phenotype-cover
- name: GreedyPC
  role: Extended-greedy phenotype cover solver; its selected features are re-derived
    on each of the 5 half-samples to estimate stability (~60-70% retention expected)
  repo: https://github.com/euxhenh/phenotype-cover
- name: CEMPC
  role: Cross-entropy-method phenotype cover solver; its random sampling nature makes
    it the expected least-stable method in the comparison
  repo: https://github.com/euxhenh/phenotype-cover
- name: mrmr
  role: Competing minimum-redundancy-maximum-relevance marker selection method (mrmr_classif)
    whose feature-set stability can be benchmarked against G-PC/CEM-PC
  repo: https://github.com/smazzanti/mrmr
- name: numpy
  role: Used for random half-sampling of cells and numpy set operations to compute
    pairwise overlap-over-union statistics between runs
- name: scikit-learn
  role: Dependency of phenotype-cover; also supplies classification/dataset utilities
    (e.g., make_classification, F values) for competing selection methods
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.crmeth.2022.100332
    title: Multiset multicover methods for discriminative marker selection
  source_content_hash: sha256:91a494bb46e14e377e9b47d1e2d3c37e01f30b46877cadc4fa9342da44d102ee
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/euxhenh/phenotype-cover
  repo_url: https://github.com/euxhenh/phenotype-cover
  tools:
  - CEMPC
  - GreedyPC
  - Python
  - mrmr
  - numpy
  - phenotype-cover
  - scikit-learn
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# marker-selection-stability-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantifies the reproducibility of feature (marker) sets selected by G-PC (greedy phenotype cover), CEM-PC (cross-entropy method), and competing marker-selection methods by repeating selection over 5 runs on random half-samples of scRNA-seq expression data and computing pairwise overlap-over-union stability indices.

## When to use

Use when you need to compare the run-to-run stability of marker sets from phenotype-cover solvers (GreedyPC/CEMPC) against global methods (ReliefF, F values, mRMR) on multi-class scRNA-seq phenotype discrimination tasks.

## When NOT to use

- When the goal is discriminative power or classification accuracy of the marker set rather than reproducibility of the selected features — use the paper's train-on-subset/logistic-regression evaluation workflow instead
- When only a single deterministic selection is needed and sampling variance is irrelevant; stability testing requires repeated runs over random half-samples (5 runs of 50% of the data)

## Inputs

- Preprocessed gene expression matrix (cells × genes) from a scRNA-seq dataset such as IPF lung (Adams et al.), mouse cortex (Zeisel et al.), or human cell atlas (He et al.), loaded via numpy/pandas
- Phenotype/class labels per cell for the marker-selection solvers
- Choice of selection method(s) to evaluate: GreedyPC, CEMPC, and/or competing methods (ReliefF, F values, mRMR)

## Outputs

- Per-method stability index aggregated from pairwise overlap-over-union statistics over 5 runs
- Fraction of features retained across runs per method (e.g., ~0.70 for G-PC)
- stability_index.csv containing the stability results table

## How to apply

Load a preprocessed expression matrix (e.g., the IPF lung, mouse cortex, or HCA scRNA-seq datasets from the phenotype-cover paper) with numpy/pandas. For each of 5 runs, randomly sample half the cells and run GreedyPC and CEMPC from the phenotype-cover package on each resample to select a marker set. For each method, compute the pairwise overlap-over-union statistic between feature sets from different runs using numpy set operations, then aggregate the pairwise statistics into a per-method stability index and record the fraction of features retained across runs. Interpretation: G-PC is less stable than global methods like ReliefF and F values due to its greedy sequential nature, but retains ~60-70% of the same genes across runs; CEM-PC is the least stable method because of its random sampling nature. Write the stability results table to stability_index.csv.

## Related tools

- **phenotype-cover** (Provides the GreedyPC and CEMPC solvers whose selected marker sets are compared across resampling runs; installed via pip install phenotype-cover (and multiset-multicover)) — https://github.com/euxhenh/phenotype-cover
- **GreedyPC** (Extended-greedy phenotype cover solver; its selected features are re-derived on each of the 5 half-samples to estimate stability (~60-70% retention expected)) — https://github.com/euxhenh/phenotype-cover
- **CEMPC** (Cross-entropy-method phenotype cover solver; its random sampling nature makes it the expected least-stable method in the comparison) — https://github.com/euxhenh/phenotype-cover
- **mrmr** (Competing minimum-redundancy-maximum-relevance marker selection method (mrmr_classif) whose feature-set stability can be benchmarked against G-PC/CEM-PC) — https://github.com/smazzanti/mrmr
- **numpy** (Used for random half-sampling of cells and numpy set operations to compute pairwise overlap-over-union statistics between runs)
- **scikit-learn** (Dependency of phenotype-cover; also supplies classification/dataset utilities (e.g., make_classification, F values) for competing selection methods)

## Examples

```
from phenotype_cover import GreedyPC; gpc = GreedyPC(); gpc.fit(X, y); features = gpc.select(100)
```

## Evaluation signals

- The computed G-PC stability index should be close to the reported ~0.70 feature retention across runs
- CEM-PC should yield the lowest stability index among all tested methods, reflecting its random sampling nature
- Global methods such as ReliefF and F values should show higher stability than G-PC, consistent with G-PC's greedy sequential nature
- stability_index.csv exists and contains one aggregated stability index per method plus the fraction of features retained
- Exactly 5 runs with half the cells sampled per run were executed, yielding C(5,2)=10 pairwise overlap-over-union statistics per method before aggregation

## Limitations

- G-PC does not provide an optimal solution; its stability is inherently lower than global methods because of its greedy sequential nature
- CEM-PC's cross-entropy random sampling makes it the least stable method, so its selected markers may vary substantially between runs
- Selection stability does not imply optimal discriminative power; markers retained across runs should still be validated for phenotype discrimination

## Evidence

- [full_text] G-PC vs. global methods stability comparison: [evidence span withheld — anchor status: not_found]
- [full_text] CEM-PC is the least stable method: [evidence span withheld — anchor status: not_found]
