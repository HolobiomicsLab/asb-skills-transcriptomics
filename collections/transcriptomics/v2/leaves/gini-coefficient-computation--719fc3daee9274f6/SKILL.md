---
name: gini-coefficient-computation
description: Use when comparing how imputation (e.g., ALRA vs. SAVER vs. raw) changes
  the gene-expression distribution of scRNA-seq data toward an orthogonal gold standard,
  via Gini coefficients over shared genes.
when_to_use_negative:
- The goal is cell-level rather than gene-distribution-level comparison (e.g., cell-type
  clustering or single-cell correlation to FISH) — Gini coefficients summarize distribution
  inequality per gene, not per cell
- The platforms share too few genes with FISH to yield a meaningful Pearson correlation
  (the FISH panel here has only 16 genes shared after filtering, 15 analyzed)
- Data are not normalized consistently across platforms (GAPDH normalization and cell
  filtering are prerequisites for comparable Gini coefficients)
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: ALRA
  role: Provides the zero-preserving low-rank-approximation imputed Drop-seq matrix
    whose gene distributions are evaluated via Gini coefficients
  repo: https://github.com/KlugerLab/ALRA
- name: rsvd
  role: Randomized SVD backend used by ALRA to compute the rank-k (k = 14) approximation
    of the Drop-seq matrix before Gini evaluation
- name: SAVER
  role: Alternative imputation method run as a baseline for the Gini-coefficient consistency
    metric (correlation ~0.6)
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-021-27729-z
    title: Zero-preserving imputation of single-cell RNA-seq data
  source_content_hash: sha256:4e3486e341ac9cfb53ddb803615b78ac78548f0423813a8eb085aeb0aba08c19
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/KlugerLab/ALRA
  tools:
  - ALRA
  - SAVER
  - rsvd
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# gini-coefficient-computation

## Summary

Computes the Gini coefficient (a measure of expression-distribution inequality) per gene in single-cell RNA-seq (Drop-seq) and RNA FISH datasets, then correlates the per-gene Gini coefficients across shared genes to quantify distribution consistency between imputation methods.

## When to use

Use when comparing how imputation (e.g., ALRA vs. SAVER vs. raw) changes the gene-expression distribution of scRNA-seq data toward an orthogonal gold standard, via Gini coefficients over shared genes.

## When NOT to use

- The goal is cell-level rather than gene-distribution-level comparison (e.g., cell-type clustering or single-cell correlation to FISH) — Gini coefficients summarize distribution inequality per gene, not per cell
- The platforms share too few genes with FISH to yield a meaningful Pearson correlation (the FISH panel here has only 16 genes shared after filtering, 15 analyzed)
- Data are not normalized consistently across platforms (GAPDH normalization and cell filtering are prerequisites for comparable Gini coefficients)

## Inputs

- GAPDH-normalized Drop-seq expression matrix (cells × genes, e.g., 8498 cells × 12,241 genes)
- GAPDH-normalized RNA FISH expression data (26 genes measured on the same melanoma cell line)
- ALRA-imputed Drop-seq matrix (rank k = 14, rsvd-based randomized SVD)
- Optional SAVER-imputed Drop-seq matrix for baseline comparison
- List of 16 genes shared between Drop-seq and FISH (GAPDH used only for normalization)

## Outputs

- Per-gene Gini coefficients for the 15 shared genes (excluding GAPDH) in FISH, raw Drop-seq, and imputed Drop-seq data
- Pearson correlations between FISH and Drop-seq Gini coefficients for raw and imputed data
- gini_fish_correlation.csv containing all correlation values

## How to apply

Start from the GAPDH-normalized expression matrices: Drop-seq (8640 cells, 32,287 genes filtered to 8498 cells, 12,241 genes; library sizes 500–20,000; bottom/top 0.1% GAPDH cells removed) and RNA FISH (26 genes, normalized by GAPDH). Restrict the comparison to the 16 genes shared between the two platforms, excluding GAPDH itself (15 genes). For each gene, compute the Gini coefficient of its expression distribution in the FISH data, the raw Drop-seq data, and the imputed Drop-seq data (ALRA-imputed, rank k = 14 via rsvd randomized SVD; optionally SAVER-imputed). Finally compute the Pearson correlation between FISH Gini coefficients and Drop-seq Gini coefficients (raw and each imputed method) over the 15 shared genes. Rationale: imputation that recovers the true expression distribution should raise the Gini-coefficient correlation above the raw baseline, and this metric is zero-preserving-sensitive because Gini inequality reflects sparsity.

## Related tools

- **ALRA** (Provides the zero-preserving low-rank-approximation imputed Drop-seq matrix whose gene distributions are evaluated via Gini coefficients) — https://github.com/KlugerLab/ALRA
- **rsvd** (Randomized SVD backend used by ALRA to compute the rank-k (k = 14) approximation of the Drop-seq matrix before Gini evaluation)
- **SAVER** (Alternative imputation method run as a baseline for the Gini-coefficient consistency metric (correlation ~0.6))

## Examples

```
cor(gini_fish, gini_dropseq_alra, method='pearson')  # 15 shared genes, result ~0.8; write to gini_fish_correlation.csv
```

## Evaluation signals

- FISH-vs-Drop-seq Gini Pearson correlation should be approximately 0.3 for raw Drop-seq data
- FISH-vs-Drop-seq Gini Pearson correlation should be approximately 0.6 for SAVER-imputed data
- FISH-vs-Drop-seq Gini Pearson correlation should be approximately 0.8 for ALRA-imputed data (the target result)
- Exactly 15 genes (shared Drop-seq/FISH genes excluding GAPDH) contribute to the correlation
- All Gini coefficients and correlations for the 15 shared genes are written to gini_fish_correlation.csv

## Limitations

- ALRA output values below the magnitude threshold are set to zero by design — methods like DCA and MAGIC output a nonzero value for every gene, so Gini comparisons to such methods are not equivalent because they treat all zeros as missing data
- Comparisons rely on only 15 shared genes, so the Pearson correlation estimate has high variance with a small gene set
- Results are sensitive to the filtering choices (library size 500–20,000, mean expression > 0.1, GAPDH-extreme-cell removal at the 0.1th/99.9th percentile)

## Evidence

- [abstract] fraction of zeros and Gini consistency with FISH: "We present a method based on low-rank matrix approximation which imputes these values while preserving biologically non-expressed genes (true biological zeros) at zero expression levels."
- [intro] imputation should preserve zeros: "it should accurately impute the data, in particular preserving at zero true biological zeros while completing the technical ones"
- [intro] zero-filling methods not comparable: "Some methods such as DCA and MAGIC treat all zeros as “missing data” and output a matrix in which every gene is expressed in every cell"
- [readme] ALRA usage and thresholding: "ALRA first computes its rank-k approximation using randomized SVD. Next, each row (gene) is thresholded by the magnitude of the most negative value of that gene."
- [readme] required input orientation and rsvd dependency: "Please be sure to pass ALRA a matrix where the cells are rows and genes are columns."
