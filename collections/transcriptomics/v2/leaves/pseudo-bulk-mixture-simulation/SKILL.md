---
name: pseudo-bulk-mixture-simulation
description: Use when you have scRNA-seq data with known per-cell type labels and
  need pseudo-bulk mixtures with ground-truth proportions to benchmark deconvolution
  performance of competing marker sets or signature matrices.
when_to_use_negative:
- You need individual-cell-level classification accuracy rather than bulk mixture
  proportion estimation — the logistic-regression left-out evaluation workflow is
  the appropriate alternative.
- You lack cell-type labels for the single cells, since ground-truth proportions for
  the mixtures cannot be recorded.
- You want truly in vivo bulk RNA-seq validation rather than simulated mixtures —
  pseudo-bulk averaging of single cells is only an approximation of real bulk tissue
  profiles.
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: CIBERSORT
  role: Performs deconvolution of pseudo-bulk mixtures using linear nu-SVR, initialized
    with nu in {0.25, 0.5, 0.75}, keeping the model with lowest RMSE
- name: GreedyPC
  role: Selects the discriminative marker set (e.g., 163 genes) from which the cell-type
    signature matrix is built
  repo: https://github.com/euxhenh/phenotype-cover
- name: numpy
  role: Numerical computation for averaging single-cell expression into pseudo-bulk
    mixtures and computing Jensen-Shannon divergence
- name: scikit-learn
  role: Provides the SVR machinery underlying the nu-SVR deconvolution step
- name: Python
  role: Implementation language for the mixture simulation and deconvolution pipeline
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.crmeth.2022.100332
    title: Multiset multicover methods for discriminative marker selection
  source_content_hash: sha256:441c949a864001b5bf874e66b9f89b45d5ad681a736b47ef352e421a6fd4d507
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
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# pseudo-bulk-mixture-simulation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Simulates bulk RNA-seq mixtures from scRNA-seq data by averaging expression across single cells in test sets, then evaluates marker sets via CIBERSORT (linear nu-SVR) deconvolution against known cell-type proportions using Jensen-Shannon divergence.

## When to use

Use when you have scRNA-seq data with known per-cell type labels and need pseudo-bulk mixtures with ground-truth proportions to benchmark deconvolution performance of competing marker sets or signature matrices.

## When NOT to use

- You need individual-cell-level classification accuracy rather than bulk mixture proportion estimation — the logistic-regression left-out evaluation workflow is the appropriate alternative.
- You lack cell-type labels for the single cells, since ground-truth proportions for the mixtures cannot be recorded.
- You want truly in vivo bulk RNA-seq validation rather than simulated mixtures — pseudo-bulk averaging of single cells is only an approximation of real bulk tissue profiles.

## Inputs

- scRNA-seq dataset with cell-type labels (e.g., IPF lung data from Adams et al., MC, or HCA datasets)
- Pre-selected marker sets per method (e.g., G-PC set of 163 genes, RankCorr set, scGeneFit set)
- Train/test split of single cells with known cell-type proportions per test set

## Outputs

- Pseudo-bulk mixture expression profiles with ground-truth cell-type proportions
- Genes x cell types signature matrix per marker-selection method
- Predicted versus true cell-type proportions per mixture
- js_divergences.csv with one row per method and an average Jensen-Shannon divergence value

## How to apply

Construct pseudo-bulk mixtures by averaging expression levels across all single cells in held-out test sets (e.g., the IPF lung scRNA-seq dataset from Adams et al.), recording the known cell-type proportions as ground truth. Build a genes x cell types signature matrix from each candidate marker set (e.g., 163-gene G-PC set vs. comparably sized RankCorr and scGeneFit sets) using average expression profiles per cell type. Run CIBERSORT deconvolution on each pseudo-bulk mixture: initialize three linear nu-SVR instances with nu in {0.25, 0.5, 0.75} and save the model achieving the lowest root-mean-square error. Compare predicted versus true cell-type proportions for every mixture and compute Jensen-Shannon divergence, then average over mixtures per method — on the IPF data this yielded G-PC JS = 0.045, RankCorr 0.056, scGeneFit 0.062.

## Related tools

- **CIBERSORT** (Performs deconvolution of pseudo-bulk mixtures using linear nu-SVR, initialized with nu in {0.25, 0.5, 0.75}, keeping the model with lowest RMSE)
- **GreedyPC** (Selects the discriminative marker set (e.g., 163 genes) from which the cell-type signature matrix is built) — https://github.com/euxhenh/phenotype-cover
- **numpy** (Numerical computation for averaging single-cell expression into pseudo-bulk mixtures and computing Jensen-Shannon divergence)
- **scikit-learn** (Provides the SVR machinery underlying the nu-SVR deconvolution step)
- **Python** (Implementation language for the mixture simulation and deconvolution pipeline)

## Examples

```
from phenotype_cover import GreedyPC; gpc = GreedyPC(); gpc.fit(X, y); genes = gpc.select(163)
```

## Evaluation signals

- Average Jensen-Shannon divergences match the reported values: G-PC JS = 0.045 at 163 genes, RankCorr 0.056, scGeneFit 0.062 on the IPF dataset
- js_divergences.csv contains one row per marker-selection method with the JS field present
- Each nu-SVR deconvolution ran with all three nu values {0.25, 0.5, 0.75} and the saved model has the lowest RMSE among the three
- Predicted proportions are non-negative and (after normalization) comparable in scale to the recorded true cell-type proportions for every mixture
- G-PC deconvolution error is lower than both RankCorr and scGeneFit at matched marker-set sizes

## Limitations

- Pseudo-bulk mixtures formed by simple averaging of single cells may not capture real bulk RNA-seq noise, dropout, or ambient RNA effects
- Results depend on the marker set size; comparisons should use comparably sized sets (e.g., 163 genes) across methods
- G-PC does not provide an optimal solution, and the deconvolution benchmark reflects both marker quality and CIBERSORT's nu-SVR behavior

## Evidence
