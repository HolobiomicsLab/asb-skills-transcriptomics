---
name: bulk-rna-seq-deconvolution
description: Use when marker-gene sets must be evaluated for how well they support
  estimating cell-type proportions from bulk or pseudo-bulk expression mixtures.
when_to_use_negative:
- The goal is selecting markers per individual cell type rather than a globally discriminatory
  set across all phenotype pairs, which is what deconvolution quality depends on
- Input is bulk data without any single-cell reference or labeled cell types from
  which to build a signature matrix
edam_operation: http://edamontology.org/operation_0314
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: CIBERSORT
  role: Performs bulk mixture deconvolution using linear nu-support vector regression
    to estimate cell-type proportions from pseudo-bulk mixtures given a signature
    matrix
- name: GreedyPC
  role: Selects the discriminative marker set (via the phenotype cover / multiset
    multicover formulation) whose deconvolution performance is being benchmarked
  repo: https://github.com/euxhenh/phenotype-cover
- name: scikit-learn
  role: Provides the SVR machinery used by the CIBERSORT-style nu-SVR deconvolution
    step
- name: numpy
  role: Supports building signature matrices and computing pseudo-bulk averages of
    single-cell expression
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.crmeth.2022.100332
    title: Multiset multicover methods for discriminative marker selection
  source_content_hash: sha256:aa44cde11498c563c7cf5c615c7d05761924d4e7a59816b24ccb9717fdf2ba12
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
  - CIBERSORT
  - GreedyPC
  - Python
  - numpy
  - scikit-learn
  edam_operation: http://edamontology.org/operation_0314
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# bulk-rna-seq-deconvolution

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark marker sets (e.g., G-PC, RankCorr, scGeneFit) for deconvolution of pseudo-bulk RNA-seq mixtures by building cell-type signature matrices, running CIBERSORT linear nu-SVR, and scoring predicted vs. true proportions with Jensen-Shannon divergence.

## When to use

Use when marker-gene sets must be evaluated for how well they support estimating cell-type proportions from bulk or pseudo-bulk expression mixtures.

## When NOT to use

- The goal is selecting markers per individual cell type rather than a globally discriminatory set across all phenotype pairs, which is what deconvolution quality depends on
- Input is bulk data without any single-cell reference or labeled cell types from which to build a signature matrix

## Inputs

- scRNA-seq dataset with cell-type labels (e.g., IPF lung dataset from Adams et al.) split into train/test sets
- Selected marker gene sets of comparable size (e.g., 163-gene G-PC, RankCorr, scGeneFit sets)
- Per-cell-type ground-truth proportions for each pseudo-bulk mixture

## Outputs

- Genes x cell types signature matrix per marker set (built from average expression profiles)
- Pseudo-bulk mixture expression profiles with known cell-type proportions
- CIBERSORT nu-SVR models (one per mixture, selected by lowest RMSE over nu in {0.25, 0.5, 0.75})
- js_divergences.csv with one row per method containing the average Jensen-Shannon divergence

## How to apply

Load the scRNA-seq dataset (e.g., the IPF lung dataset from Adams et al.) together with comparably sized marker sets such as the G-PC (greedy phenotype cover) set of 163 genes, RankCorr, and scGeneFit. Build a genes x cell types signature matrix from each set using average expression profiles per cell type. Construct pseudo-bulk mixtures by averaging expression levels across all single cells in the test sets, recording known cell-type proportions as ground truth. Run CIBERSORT deconvolution, which uses linear nu-support vector regression, by initializing three nu-SVR instances with nu in {0.25, 0.5, 0.75} and saving the model that achieves the lowest root-mean-square error. Compare predicted versus true proportions for every mixture, compute the Jensen-Shannon divergence per mixture, and average over mixtures per method; on the IPF dataset at 163 genes the expected ordering is G-PC JS = 0.045, RankCorr 0.056, scGeneFit 0.062.

## Related tools

- **CIBERSORT** (Performs bulk mixture deconvolution using linear nu-support vector regression to estimate cell-type proportions from pseudo-bulk mixtures given a signature matrix)
- **GreedyPC** (Selects the discriminative marker set (via the phenotype cover / multiset multicover formulation) whose deconvolution performance is being benchmarked) — https://github.com/euxhenh/phenotype-cover
- **scikit-learn** (Provides the SVR machinery used by the CIBERSORT-style nu-SVR deconvolution step)
- **numpy** (Supports building signature matrices and computing pseudo-bulk averages of single-cell expression)

## Examples

```
from phenotype_cover import GreedyPC; gpc = GreedyPC(); gpc.fit(X, y); features = gpc.select(163)
```

## Evaluation signals

- Average Jensen-Shannon divergence on the IPF dataset at 163 genes matches reported values: G-PC 0.045, RankCorr 0.056, scGeneFit 0.062
- js_divergences.csv contains one row per marker-selection method, each with a JS field present
- For every mixture, the saved nu-SVR model is the one (from nu = 0.25, 0.5, 0.75) with the lowest root-mean-square error
- Each signature matrix contains only the genes of the corresponding marker set, with one column per cell type
- Predicted proportions are non-negative and comparable in dimensionality to the recorded true proportions per mixture

## Limitations

- G-PC does not provide an optimal solution to the multiset multicover (NP-complete) problem, so deconvolution results are those of a polynomial-time approximation
- Pseudo-bulk mixtures built by averaging single-cell expression are a simplification of real bulk RNA-seq and may not capture real mixture noise
- Biological interpretation of selected genes relied on overlap with existing marker lists that may be incomplete

## Evidence

- [results] Pseudo-bulk mixture construction from test sets: "we constructed pseudo-bulk mixtures using the IPF, MC, and HCA datasets by averaging expression levels across all single cells in the test sets"
- [results] nu-SVR initialization and model selection: "we initialize three linear ν -SVR instances with ν∈{0.25,0.5,0.75} and save the model that achieves the lowest root-mean-square error"
- [results] CIBERSORT as the deconvolution method: "One widely used method for deconvolution is CIBERSORT,15 which uses ν -support vector regression (ν -SVR)."
- [intro] Deconvolution depends on discriminatory markers across all types: "Deconvolution of cell types from bulk data is also highly dependent on the ability to select not just good markers for each individual cell type but also a set of discriminatory markers between all"
- [readme] GreedyPC marker selection usage: ">>> gpc = GreedyPC()
    >>> gpc.fit(X, y)
    >>> features = gpc.select(100)  # coverage of 100"
