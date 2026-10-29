---
name: single-cell-crispr-screen-association-testing
description: Use when analyzing UMI read count + gRNA assignment matrices from a pooled
  single-cell CRISPR screen (e.g., high-MOI CROP-seq/Perturb-seq with thousands of
  gRNAs) to compute unbiased gRNA-gene associations with logFC, P values, and Q values.
when_to_use_negative:
- Hypothesis testing of arbitrary nonlinear associations — Normalisr is explicitly
  not designed for this
- Full-length (non-UMI) scRNA-seq platforms — the method was only validated on UMI-based
  datasets
- Reconstructing causal regulatory networks (v-structures, regularized regression,
  Granger causality) — Normalisr only provides linear association testing
- Integrative analyses spanning multiple datasets with batch effects — not addressed
  by Normalisr
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3168
tools:
- name: Normalisr
  role: 'Primary tool: performs quality control, library-size-confounder normalization,
    and linear-model gRNA-gene association testing with exact P values (no permutation)
    and native covariate removal'
  repo: https://github.com/lingfeiwang/normalisr
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-021-26682-1
    title: Single-cell normalization and association testing unifying CRISPR screen
      and gene co-expression analyses with Normalisr
  source_content_hash: sha256:78e5c0b35d1dce1ea5f374b6e64545bf72186da5d18f9530b549468812206ff0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lingfeiwang/normalisr
  tools:
  - Normalisr
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# single-cell-crispr-screen-association-testing

## Summary

Association testing of gRNA-gene perturbation effects in high-MOI single-cell CRISPR screens using Normalisr's normalized linear-model framework, including competition-aware adjustment that includes untested gRNAs as covariates to control false positive rate inflation.

## When to use

Use when analyzing UMI read count + gRNA assignment matrices from a pooled single-cell CRISPR screen (e.g., high-MOI CROP-seq/Perturb-seq with thousands of gRNAs) to compute unbiased gRNA-gene associations with logFC, P values, and Q values.

## When NOT to use

- Hypothesis testing of arbitrary nonlinear associations — Normalisr is explicitly not designed for this
- Full-length (non-UMI) scRNA-seq platforms — the method was only validated on UMI-based datasets
- Reconstructing causal regulatory networks (v-structures, regularized regression, Granger causality) — Normalisr only provides linear association testing
- Integrative analyses spanning multiple datasets with batch effects — not addressed by Normalisr

## Inputs

- UMI read count matrix (tsv or Cell Ranger sparse mtx, gzipped accepted)
- gRNA assignment matrix (tsv)
- Row/column name text files (cells and genes, one per line)

## Outputs

- gRNA-gene association table with log fold-changes, P values, and Q values for every pair (e.g., de_associations.tsv)
- False positive rate estimates via Storey's method on NTC P-value histograms (naive vs competition-aware)
- Association-inflation ratio at Q<=0.2

## How to apply

Download the UMI read count matrix and gRNA assignment matrix (e.g., the 207,324-cell K562 full-scale CRISPRi screen with 13,186 gRNAs and 10,877 genes from GEO). Preprocess with Normalisr's quality control (qc_reads, qc_outlier) and normalization pipeline (lcpm, normcov, scaling_factor, compute_var, normvar), which systematically removes nonlinear library-size confounders at both mean and variance levels. Run the 'de' submodule as multivariate differential expression across gRNA-gene pairs; include untested gRNAs as fixed-effect covariates (competition-aware configuration), reducing them to the top-500 principal components when their count exceeds 10,000. Compare against the naive (no untested-gRNA covariates) run to quantify FPR inflation: estimate FPR by Storey's method on P-value histograms of non-targeting-control gRNAs (naive ~4.8% vs competition-aware ~1.5% overall). Extract log fold-changes, P values, and Q values for every gRNA-gene pair and write the association table (e.g., de_associations.tsv).

## Related tools

- **Normalisr** (Primary tool: performs quality control, library-size-confounder normalization, and linear-model gRNA-gene association testing with exact P values (no permutation) and native covariate removal) — https://github.com/lingfeiwang/normalisr

## Examples

```
pip install normalisr && normalisr de -h
```

## Evaluation signals

- NTC gRNA P-value histogram is near-uniform; Storey-estimated overall FPR is ~1.5% for the competition-aware run versus ~4.8% for the naive run
- Competition-aware run includes untested gRNAs as fixed-effect covariates (top-500 principal components when >10,000 untested gRNAs)
- Output table contains log fold-change, P value, and Q value for every gRNA-gene pair (13,186 x 10,877 pairs for the full-scale K562 screen)
- P values are computed exactly via linear models without permutation and remain unbiased across cells
- Association-inflation ratio computed at Q<=0.2 shows no inflation relative to naive analysis

## Limitations

- Normalisr is not designed for hypothesis testing of arbitrary nonlinear associations
- Only UMI-based datasets were analyzed; full-length platforms not validated and may require revalidation of the nonlinear cellular summary covariates
- Does not reconstruct causal regulatory networks or Granger causality
- Batch effects in multi-dataset integrative studies are not addressed

## Evidence

- [abstract] Here we present Normalisr, a normalization and statistical association testing framework that unifies single-cell differential expression, co-expression, and CRISPR screen analyses with linear models.: "Here we present Normalisr, a normalization and statistical association testing framework that unifies single-cell differential expression, co-expression, and CRISPR screen analyses with linear models."
- [readme] Normalisr first removes confounding technical noises from raw read counts to recover the biological variations. Then, linear association testing provides a unified inferential framework with several advantages: "Normalisr first removes confounding technical noises from raw read counts to recover the biological variations. Then, linear association testing provides a unified inferential framework with several"
- [readme] Type ``normalisr`` or ``normalisr -h`` for general help, and for example ``normalisr de -h`` for help on submodule 'de' of differential expression.: [evidence span withheld — anchor status: external_doc]
- [abstract] By systematically detecting and removing nonlinear confounders arising from library size at mean and variance levels, Normalisr achieves high sensitivity, specificity, speed, and generalizability: "By systematically detecting and removing nonlinear confounders arising from library size at mean and variance levels"
