---
name: drop-seq-fish-distribution-comparison
description: Use when evaluating an imputation method on Drop-seq data against RNA
  FISH measurements of the same cell line, using Gini-coefficient correlation across
  shared genes as the agreement metric.
when_to_use_negative:
- The FISH and scRNA-seq datasets share too few genes or lack a common normalization
  gene (GAPDH) for aligned per-gene Gini comparison
- The goal is to evaluate imputation accuracy on simulated data or by correlation
  of imputed vs. held-out values rather than distribution-level consistency
- The data are not droplet-based scRNA-seq (the >90% zero fraction motivating imputation
  is specific to droplet techniques)
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3523
- http://edamontology.org/topic_3308
tools:
- name: ALRA
  role: Primary imputation method whose output (ALRA-imputed Drop-seq matrix, rank
    k=14, rsvd randomized SVD) is scored for Gini-coefficient consistency with FISH
  repo: https://github.com/KlugerLab/ALRA
- name: rsvd
  role: Randomized SVD package used by ALRA for the rank-k approximation of the normalized
    Drop-seq matrix
  repo: https://github.com/KlugerLab/ALRA
- name: SAVER
  role: Baseline imputation method whose Gini-correlation performance (~0.6) is compared
    against ALRA (~0.8) and raw Drop-seq (~0.3)
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-021-27729-z
    title: Zero-preserving imputation of single-cell RNA-seq data
  source_content_hash: sha256:3051caf553efd7eeb0bc037fa16a918ec2b83583d25c3aabc1189b1e47c59ce0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/KlugerLab/ALRA
  tools:
  - ALRA
  - SAVER
  - rsvd
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3523
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# drop-seq-fish-distribution-comparison

## Summary

A benchmark that quantifies whether scRNA-seq imputation (e.g., ALRA) makes the distribution of single-cell gene expression values more consistent with RNA FISH measurements, by correlating per-gene Gini coefficients computed on FISH versus Drop-seq data.

## When to use

Use when evaluating an imputation method on Drop-seq data against RNA FISH measurements of the same cell line, using Gini-coefficient correlation across shared genes as the agreement metric.

## When NOT to use

- The FISH and scRNA-seq datasets share too few genes or lack a common normalization gene (GAPDH) for aligned per-gene Gini comparison
- The goal is to evaluate imputation accuracy on simulated data or by correlation of imputed vs. held-out values rather than distribution-level consistency
- The data are not droplet-based scRNA-seq (the >90% zero fraction motivating imputation is specific to droplet techniques)

## Inputs

- Drop-seq expression matrix (8640 cells, 32,287 genes) from Torre et al. (2018)
- RNA FISH measurements (26 genes) of the same melanoma cell line from Torre et al. (2018)
- Optionally, SAVER-imputed and ALRA-imputed Drop-seq matrices for baseline comparison

## Outputs

- gini_fish_correlation.csv containing FISH-vs-Drop-seq Gini Pearson correlations for the 15 shared genes (raw and imputed)
- Per-gene Gini coefficients for FISH, raw Drop-seq, and ALRA-imputed Drop-seq

## How to apply

Obtain the Drop-seq matrix (8640 cells, 32,287 genes) and RNA FISH data (26 genes) from Torre et al. (2018). Filter cells with library size >20,000 or <500 and genes with mean expression <0.1, yielding 8498 cells, 12,241 genes, and 16 genes shared between the platforms. Remove cells in the bottom and top tenth percentiles of GAPDH expression, then normalize all genes by GAPDH expression. Run ALRA on the normalized Drop-seq matrix with rank k=14 using rsvd-based randomized SVD, then compute the Gini coefficient of each of the 15 shared genes (excluding GAPDH) in the FISH data, the raw Drop-seq data, and the ALRA-imputed data. Compute the Pearson correlation between FISH Gini coefficients and Drop-seq Gini coefficients for the 15 shared genes; the rationale is that imputation should make single-cell expression distributions match the FISH-measured distribution of the same transcripts.

## Related tools

- **ALRA** (Primary imputation method whose output (ALRA-imputed Drop-seq matrix, rank k=14, rsvd randomized SVD) is scored for Gini-coefficient consistency with FISH) — https://github.com/KlugerLab/ALRA
- **rsvd** (Randomized SVD package used by ALRA for the rank-k approximation of the normalized Drop-seq matrix) — https://github.com/KlugerLab/ALRA
- **SAVER** (Baseline imputation method whose Gini-correlation performance (~0.6) is compared against ALRA (~0.8) and raw Drop-seq (~0.3))

## Examples

```
library(ALRA); A_norm <- normalize_data(b_nk_example); A_norm_completed <- alra(A_norm, k=14)[[3]]
```

## Evaluation signals

- FISH-vs-ALRA-imputed Drop-seq Gini Pearson correlation should be approximately 0.8
- FISH-vs-raw-Drop-seq Gini Pearson correlation should be approximately 0.3
- FISH-vs-SAVER-imputed Drop-seq Gini Pearson correlation should be approximately 0.6
- After filtering: 8498 cells, 12,241 genes, and 16 genes shared between Drop-seq and FISH; 15 genes (excluding GAPDH) enter the correlation
- All correlation values are written to gini_fish_correlation.csv

## Limitations

- Only 15 shared genes (excluding GAPDH) are available for the correlation, so the metric rests on very few points
- ALRA's parameter k must be specified (k=14 here); the README notes k is chosen from singular value spacings and 'in some cases you might want to take a closer look'
- Raw Drop-seq zeros can exceed 90% of the matrix with droplet-based techniques, making distribution-level comparisons sensitive to dropout rather than biology

## Evidence

- [abstract] imputation: "We present a method based on low-rank matrix approximation which imputes these values while preserving biologically non-expressed genes (true biological zeros) at zero expression levels."
- [methods] rsvd-k14: [evidence span withheld — anchor status: not_found]
- [results] validation-thresholds: [evidence span withheld — anchor status: not_found]
- [readme] alra-usage: "The only prerequisite for ALRA is installation of the randomized SVD package RSVD which can be installed as `install.packages('rsvd')`."
