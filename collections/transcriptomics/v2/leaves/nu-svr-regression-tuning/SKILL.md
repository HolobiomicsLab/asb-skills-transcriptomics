---
name: nu-svr-regression-tuning
description: Use when deconvolving pseudo-bulk or bulk RNA-seq mixtures with a signature
  matrix (e.g., built from G-PC/RankCorr/scGeneFit markers) and you need to fit cell-type
  fractions with CIBERSORT's linear nu-SVR approach.
when_to_use_negative:
- Input is single-cell expression data that has not yet been aggregated into pseudo-bulk
  mixtures with known proportions
- The task is marker selection itself (use phenotype-cover/GreedyPC rather than deconvolution);
  this skill assumes a marker set already exists
- No ground-truth cell-type proportions are available, so JS divergence between predicted
  and true proportions cannot be computed
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: CIBERSORT
  role: Deconvolution framework whose nu-SVR regression this skill instantiates and
    tunes
- name: scikit-learn
  role: Provides linear nu-SVR implementation used to fit the three nu values
- name: numpy
  role: Array manipulation for signature matrices and mixture averaging
- name: phenotype-cover
  role: Supplies the G-PC marker set evaluated via this deconvolution workflow
  repo: https://github.com/euxhenh/phenotype-cover
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.crmeth.2022.100332
    title: Multiset multicover methods for discriminative marker selection
  source_content_hash: sha256:b29cb441f09fa09e41d9bf8aa0d0969f8be83bd9453eaafcd45dcdde3c6e380a
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
  - phenotype-cover
  - scikit-learn
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# nu-svr-regression-tuning

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Tunes linear nu-SVR for CIBERSORT-style cell-type proportion deconvolution by initializing three nu-SVR instances with nu in {0.25, 0.5, 0.75} and keeping the model with the lowest root-mean-square error, then scoring predicted vs. true proportions with Jensen-Shannon divergence.

## When to use

Use when deconvolving pseudo-bulk or bulk RNA-seq mixtures with a signature matrix (e.g., built from G-PC/RankCorr/scGeneFit markers) and you need to fit cell-type fractions with CIBERSORT's linear nu-SVR approach.

## When NOT to use

- Input is single-cell expression data that has not yet been aggregated into pseudo-bulk mixtures with known proportions
- The task is marker selection itself (use phenotype-cover/GreedyPC rather than deconvolution); this skill assumes a marker set already exists
- No ground-truth cell-type proportions are available, so JS divergence between predicted and true proportions cannot be computed

## Inputs

- Cell-type signature matrix (genes x cell types) built from average expression profiles of a selected marker set (e.g., 163 G-PC genes on the IPF dataset)
- Pseudo-bulk mixture expression vectors obtained by averaging single-cell expression in the IPF (Adams et al.) test sets
- Known (ground-truth) cell-type proportions for each mixture

## Outputs

- Fitted linear nu-SVR deconvolution model per mixture (the nu in {0.25, 0.5, 0.75} instance with lowest RMSE)
- Predicted cell-type proportions per pseudo-bulk mixture
- Per-method average Jensen-Shannon divergence (e.g., js_divergences.csv with one row per method and a JS field)

## How to apply

Build a genes x cell types signature matrix from average expression per cell type over the selected marker set, and construct pseudo-bulk mixtures by averaging expression across all single cells in the test set while recording known proportions as ground truth. For each mixture, initialize three linear nu-SVR instances with nu in {0.25, 0.5, 0.75} and save the model that achieves the lowest root-mean-square error between observed and signature-derived expression. Compare predicted versus true cell-type proportions and compute the Jensen-Shannon divergence per mixture, then average over mixtures. Rationale: deconvolution depends on discriminatory markers between all cell types, and this paper's marker sets (e.g., G-PC at 163 genes) are evaluated through this exact nu-SVR tuning loop on the IPF dataset, yielding G-PC JS = 0.045 vs. RankCorr 0.056 and scGeneFit 0.062.

## Related tools

- **CIBERSORT** (Deconvolution framework whose nu-SVR regression this skill instantiates and tunes)
- **scikit-learn** (Provides linear nu-SVR implementation used to fit the three nu values)
- **numpy** (Array manipulation for signature matrices and mixture averaging)
- **phenotype-cover** (Supplies the G-PC marker set evaluated via this deconvolution workflow) — https://github.com/euxhenh/phenotype-cover

## Examples

```
from sklearn.svm import NuSVR; best = min([NuSVR(nu=v, kernel='linear').fit(sig.T, mix) for v in (0.25, 0.5, 0.75)], key=lambda m: rmse(m, sig, mix))
```

## Evaluation signals

- Average Jensen-Shannon divergence matches the reported values on the IPF dataset: G-PC 0.045 at 163 genes, RankCorr 0.056, scGeneFit 0.062
- js_divergences.csv contains one row per marker-selection method with the JS field present
- Exactly one of the three nu-SVR instances (nu in {0.25, 0.5, 0.75}) is saved per mixture, and it is the one with lowest RMSE
- Predicted proportions per mixture are non-negative and sum to approximately 1
- The nu-SVR instances use a linear kernel as specified in the CIBERSORT workflow

## Limitations

- G-PC (and the overall workflow) does not guarantee optimality; the marker set feeding deconvolution is an approximation to the phenotype cover problem
- Results are tied to specific datasets (IPF, mouse cortex, human cell atlas); JS divergences may differ on other tissues or batch-affected data (the paper tested batch effects with two pancreas datasets)
- Only three nu values are searched; finer grids or other regressors might yield lower RMSE

## Evidence
