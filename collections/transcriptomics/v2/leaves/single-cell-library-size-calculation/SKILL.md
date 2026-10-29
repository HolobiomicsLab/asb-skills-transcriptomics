---
name: single-cell-library-size-calculation
description: Use when preprocessing raw scRNA-seq count matrices (e.g., iasva_counts_test.Rds)
  and you need to decide which principal components or library-size covariates capture
  technical depth variation before surrogate-variable analysis.
when_to_use_negative:
- Input is already normalized/size-factor-corrected expression (e.g., SCnorm- or SCRAN-normalized
  values), where colSums(log(counts+1)) no longer represents sequencing depth
- Analysis goal is to detect hidden biological factors rather than to quantify/adjust
  for library-size-driven technical variation
- The matrix is too sparse/small for a meaningful PCA (e.g., lowly expressed genes
  retained without applying expression filters)
edam_operation: http://edamontology.org/operation_3465
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: irlba
  role: Computes PC1 (first right singular vector) of the column-centered log-count
    matrix for correlation with geometric library size
  repo: https://bwlewis.github.io/irlba/
- name: iasva
  role: Provides the alpha-cell count matrix (iasva_counts_test.Rds) and downstream
    IA-SVA framework where geometric library size is used as a known covariate
  repo: https://github.com/UcarLab/iasva
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41598-018-35365-9
    title: Detection of correlated hidden factors from single cell transcriptomes
      using Iteratively Adjusted-SVA (IA-SVA)
  source_content_hash: sha256:d05f4ff92b8fcefaae19b9066d0b515a722acf22551a443c8638dddb1b722f7a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - IRLBA
  - iasva
  - irlba
  edam_operation: http://edamontology.org/operation_3465
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# single-cell-library-size-calculation

## Summary

Compute the geometric library size (colSums of log-transformed counts) for a scRNA-seq cell matrix and quantify how strongly it dominates variability by correlating it with the first principal component (PC1) of the log-transformed matrix. In the IA-SVA workflow this established that PC1 on 101 Lawlor alpha cells was essentially library size (r = 0.99), justifying geometric library size as a known covariate.

## When to use

Use when preprocessing raw scRNA-seq count matrices (e.g., iasva_counts_test.Rds) and you need to decide which principal components or library-size covariates capture technical depth variation before surrogate-variable analysis.

## When NOT to use

- Input is already normalized/size-factor-corrected expression (e.g., SCnorm- or SCRAN-normalized values), where colSums(log(counts+1)) no longer represents sequencing depth
- Analysis goal is to detect hidden biological factors rather than to quantify/adjust for library-size-driven technical variation
- The matrix is too sparse/small for a meaningful PCA (e.g., lowly expressed genes retained without applying expression filters)

## Inputs

- Raw gene-by-cell scRNA-seq read count matrix (e.g., iasva_counts_test.Rds, alpha cells n = 101 from Lawlor et al. 2016, Fluidigm C1)

## Outputs

- Per-cell geometric library size vector: geo_lib_size = colSums(log(counts + 1))
- PC1 score vector (first right singular vector of the column-centered log-count matrix via irlba)
- Pearson correlation coefficient r between geometric library size and PC1 (e.g., written to correlation_rslts.csv)

## How to apply

Load the raw gene-by-cell read count matrix (e.g., the 101 Lawlor et al. alpha-cell matrix iasva_counts_test.Rds shipped with the iasva package). Compute geometric library size per cell as colSums(log(counts + 1)). Log-transform the counts (lcounts = log(counts + 1)), column-center them by subtracting rowMeans, and extract PC1 as the first right singular vector via the irlba package (e.g., irlba(lcounts - rowMeans(lcounts), 1)$v[, 1]). Compute the Pearson correlation between the geometric library size vector and PC1 with cor(). In the IA-SVA paper this yielded r = 0.99 on the alpha cells, so PC1 was discarded from the PCA, and geometric library size (together with patient identity) was carried forward as a known covariate for IA-SVA, USVA and SSVA. Write the resulting correlation value (e.g., to correlation_rslts.csv) to document the decision.

## Related tools

- **irlba** (Computes PC1 (first right singular vector) of the column-centered log-count matrix for correlation with geometric library size) — https://bwlewis.github.io/irlba/
- **iasva** (Provides the alpha-cell count matrix (iasva_counts_test.Rds) and downstream IA-SVA framework where geometric library size is used as a known covariate) — https://github.com/UcarLab/iasva

## Examples

```
R: geo_lib_size <- colSums(log(counts + 1)); lcounts <- log(counts + 1); pc1 <- irlba::irlba(lcounts - rowMeans(lcounts), 1)$v[, 1]; r <- cor(geo_lib_size, pc1); write.csv(data.frame(r), "correlation_rslts.csv")
```

## Evaluation signals

- The Pearson correlation between geometric library size and PC1 should be very high (article reports r = 0.99 on the 101 alpha cells; workflow validation expects approximately r ~ 0.9)
- geo_lib_size vector length must equal the number of cells (n = 101 for iasva_counts_test.Rds) and be strictly positive
- PC1 must be extracted as the first right singular vector of the column-centered log-count matrix (lcounts - rowMeans(lcounts)), not of the raw counts
- The correlation value is persisted to correlation_rslts.csv so the PC1-discarding decision is auditable
- Downstream: geometric library size (with patient identity) appears as a covariate for IA-SVA/USVA/SSVA but not for PCA

## Limitations

- PC1-based library-size correlation is estimated on log(counts+1); other normalization schemes (e.g., SCnorm, SCRAN) change the interpretation of the quantity
- This skill only quantifies/adjusts for depth-related variation; it does not estimate hidden correlated factors — IA-SVA itself is required for that
- The paper does not specify a correlation cutoff above which PC1 must be discarded; the r = 0.99 discard decision was made ad hoc for the alpha-cell data

## Evidence

- [results] PC1 discarded as library size: "PC1 was discarded since it is highly correlated (r = 0.99) with the geometric library size"
- [results] Library size as known covariate: [evidence span withheld — anchor status: not_found]
- [methods] Data source for the skill: "we use read counts of alpha cells from healthy (non-diabetic) subjects (n = 101)"
- [readme] Package installation providing the test data: [evidence span withheld — anchor status: external_doc]
- [readme] IA-SVA adjusts for known factors while uncovering hidden ones: "identify a hidden factor for unwanted heterogeneity while adjusting for all known factors"
