---
name: jensen-shannon-divergence-proportion-evaluation
description: Use when you have deconvolution-predicted cell-type proportions (e.g.,
  from CIBERSORT nu-SVR runs) and known true proportions for pseudo-bulk mixtures,
  and need a single distributional metric to rank marker sets.
when_to_use_negative:
- You lack ground-truth cell-type proportions for the mixtures — JS comparison against
  true proportions is impossible and a different validation strategy is needed
- You need to select markers rather than evaluate an already-selected marker set —
  use the phenotype cover / G-PC selection skill instead
- Predictions are continuous labels or class scores rather than compositional proportion
  vectors — JS divergence over cell-type fractions is the wrong metric
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: CIBERSORT
  role: Deconvolution method producing the predicted cell-type proportions via linear
    nu-SVR that are compared to truth using JS divergence
- name: scikit-learn
  role: Provides the linear nu-SVR regressors (NuSVR with nu in {0.25, 0.5, 0.75})
    used for the deconvolution step feeding this evaluation
- name: GreedyPC / phenotype-cover
  role: Selects the G-PC marker sets whose deconvolution performance (average JS divergence)
    is quantified by this skill
  repo: https://github.com/euxhenh/phenotype-cover
- name: numpy
  role: Numerical dependency of phenotype-cover used for computing divergences and
    proportion vectors
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.crmeth.2022.100332
    title: Multiset multicover methods for discriminative marker selection
  source_content_hash: sha256:ef65b0e3e2dd8230311adf9ba8910b740e35c0b09ddf5631ce6c3eef42895af2
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
  - GreedyPC / phenotype-cover
  - Python
  - numpy
  - scikit-learn
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# jensen-shannon-divergence-proportion-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Evaluates deconvolution-derived cell-type proportion estimates against known ground-truth proportions by computing the Jensen-Shannon divergence per pseudo-bulk mixture and averaging per marker-selection method, as done for CIBERSORT runs on IPF, MC, and HCA scRNA-seq pseudo-bulk mixtures.

## When to use

Use when you have deconvolution-predicted cell-type proportions (e.g., from CIBERSORT nu-SVR runs) and known true proportions for pseudo-bulk mixtures, and need a single distributional metric to rank marker sets.

## When NOT to use

- You lack ground-truth cell-type proportions for the mixtures — JS comparison against true proportions is impossible and a different validation strategy is needed
- You need to select markers rather than evaluate an already-selected marker set — use the phenotype cover / G-PC selection skill instead
- Predictions are continuous labels or class scores rather than compositional proportion vectors — JS divergence over cell-type fractions is the wrong metric

## Inputs

- Pseudo-bulk mixture expression profiles (averaged across single cells in test sets from IPF/MC/HCA scRNA-seq data)
- Ground-truth cell-type proportion vectors for each mixture (recorded from known single-cell labels)
- Predicted cell-type proportion vectors from CIBERSORT linear nu-SVR deconvolution (nu in {0.25, 0.5, 0.75}, best-RMSE model saved)
- Gene x cell type signature matrix built from each marker set (G-PC 163 genes, RankCorr, scGeneFit)

## Outputs

- Per-mixture Jensen-Shannon divergence between predicted and true cell-type proportions
- Average JS divergence per marker-selection method (e.g., js_divergences.csv with one row per method and a JS field)
- Method ranking (G-PC 0.045 < RankCorr 0.056 < scGeneFit 0.062 on IPF at 163 genes)

## How to apply

Construct pseudo-bulk mixtures by averaging expression levels across all single cells in the test sets, recording known cell-type proportions as ground truth. Run deconvolution with CIBERSORT-style linear nu-SVR on each mixture against a signature matrix built from each marker set (e.g., a 163-gene G-PC set and comparably sized RankCorr and scGeneFit sets), initializing three nu-SVR instances with nu in {0.25, 0.5, 0.75} and saving the model achieving the lowest root-mean-square error. For every mixture, compare predicted versus true proportions via the Jensen-Shannon divergence, then average over mixtures per method. On the IPF dataset (Adams et al.), this procedure yields average JS divergences of 0.045 for G-PC, 0.056 for RankCorr, and 0.062 for scGeneFit at 163 genes, so lower JS indicates a more discriminative marker set.

## Related tools

- **CIBERSORT** (Deconvolution method producing the predicted cell-type proportions via linear nu-SVR that are compared to truth using JS divergence)
- **scikit-learn** (Provides the linear nu-SVR regressors (NuSVR with nu in {0.25, 0.5, 0.75}) used for the deconvolution step feeding this evaluation)
- **GreedyPC / phenotype-cover** (Selects the G-PC marker sets whose deconvolution performance (average JS divergence) is quantified by this skill) — https://github.com/euxhenh/phenotype-cover
- **numpy** (Numerical dependency of phenotype-cover used for computing divergences and proportion vectors)

## Examples

```
from sklearn.svm import NuSVR; models = [NuSVR(nu=n, kernel='linear').fit(signature, mixture) for n in (0.25, 0.5, 0.75)]; best = min(models, key=lambda m: rmse)
```

## Evaluation signals

- Computed average JS divergences match reported values on IPF: G-PC 0.045, RankCorr 0.056, scGeneFit 0.062 at 163 genes
- js_divergences.csv contains exactly one row per marker-selection method, each with a non-null JS field
- For each mixture, the saved nu-SVR model is the one with the lowest RMSE among nu in {0.25, 0.5, 0.75}
- Identical predicted and true proportion vectors give JS divergence of 0; divergences are non-negative and bounded (0 for identical distributions)
- G-PC's average JS divergence is lower than both RankCorr's and scGeneFit's, reproducing the reported ordering

## Limitations

- G-PC does not provide an optimal solution, so the winning marker set (and its JS score) is an approximation-quality result, not a guaranteed optimum
- JS-divergence ranking is specific to pseudo-bulk mixtures built by averaging all single cells in test sets; other mixture constructions may change the ordering
- Biological analysis relied on overlap with existing marker lists that may be incomplete, limiting interpretation beyond the numeric divergence comparison

## Evidence
