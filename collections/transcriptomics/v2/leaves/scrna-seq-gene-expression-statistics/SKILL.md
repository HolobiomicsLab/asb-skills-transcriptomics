---
name: scrna-seq-gene-expression-statistics
description: Use when you have a raw gene-by-cell count matrix (e.g. GEMLI_example_data_matrix.RData)
  and need to reproduce or audit how GEMLI selects candidate gene-expression-memory
  genes via mean expression and CV2 variability.
when_to_use_negative:
- Input is normalized or size-factor-corrected expression rather than raw counts,
  since CV2 and its mean dependence are defined on raw count variance
- The goal is downstream lineage prediction, cell-fate DEG calling, or network trimming
  (use predict_lineages, cell_fate_DEG_calling, or trim_network_to_size instead of
  this gene-selection step)
- The cell population contains only distantly related cells, where GEMLI's underlying
  gene-expression-memory signal performs less robustly
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: GEMLI
  role: R package providing the GEMLI items list and gene_expression slot whose memory-gene
    selection criterion is being reconstructed; installed from UPSUTER/GEMLI
  repo: https://github.com/UPSUTER/GEMLI
- name: R
  role: Compute per-gene means, CV2, CV2-on-mean regression residuals, quantiles,
    and export the selected gene set to CSV
- name: devtools
  role: Used to install the GEMLI package from GitHub via install_github("UPSUTER/GEMLI",
    subdir="GEMLI_package_v0")
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-024-47158-y
    title: Gene-expression memory-based prediction of cell lineages from scRNA-seq
      datasets
  source_content_hash: sha256:f48a7ece0bd51648f24be608242687801bcbea810f06c9295007e4a1e97cd63d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/UPSUTER/GEMLI
  tools:
  - GEMLI
  - R
  - devtools
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# scrna-seq-gene-expression-statistics

## Summary

Reconstructs GEMLI's initial memory-gene selection criterion from a raw scRNA-seq count matrix by ranking genes on mean expression and mean-corrected CV2 variability, then applying the paper's quantile-based selection rule to recover the memory-gene-enriched gene set.

## When to use

Use when you have a raw gene-by-cell count matrix (e.g. GEMLI_example_data_matrix.RData) and need to reproduce or audit how GEMLI selects candidate gene-expression-memory genes via mean expression and CV2 variability.

## When NOT to use

- Input is normalized or size-factor-corrected expression rather than raw counts, since CV2 and its mean dependence are defined on raw count variance
- The goal is downstream lineage prediction, cell-fate DEG calling, or network trimming (use predict_lineages, cell_fate_DEG_calling, or trim_network_to_size instead of this gene-selection step)
- The cell population contains only distantly related cells, where GEMLI's underlying gene-expression-memory signal performs less robustly

## Inputs

- Raw gene-by-cell count matrix from GEMLI_example_data_matrix.RData (RData format, rows = genes, columns = cells)
- List of memory genes highlighted in Fig. 2b of the paper (for enrichment validation)

## Outputs

- selected_memory_genes.csv containing the selected gene identifiers
- Per-gene mean expression values, CV2 values, and CV2-on-mean regression residuals
- Quantile-transformed mean and variability scores per gene

## How to apply

Load the raw count matrix (rows = genes, columns = cells) from GEMLI_example_data_matrix.RData into the 'gene_expression' slot of a GEMLI items list in R. Compute per-gene mean expression and squared coefficient of variation (CV2), then regress CV2 on mean expression and take the residuals as the mean-corrected variability measure, so that selection is not biased toward highly expressed genes. Convert mean expression and residual variability to quantiles across all genes and keep genes meeting any of: mean quantile >= 98, OR (mean quantile >= 90 AND variability quantile >= 40), OR (mean quantile >= 60 AND variability quantile >= 90). Export the surviving gene identifiers to selected_memory_genes.csv and validate by checking that this set enriches for the memory genes highlighted in Fig. 2b of the Eisele et al. (2024) paper rather than a random gene subset. The rationale is that GEMLI exploits genes with expression stable across cell generations, so the initial gene filter must capture the high-expression and high-residual-variability population where memory genes reside.

## Related tools

- **GEMLI** (R package providing the GEMLI items list and gene_expression slot whose memory-gene selection criterion is being reconstructed; installed from UPSUTER/GEMLI) — https://github.com/UPSUTER/GEMLI
- **R** (Compute per-gene means, CV2, CV2-on-mean regression residuals, quantiles, and export the selected gene set to CSV)
- **devtools** (Used to install the GEMLI package from GitHub via install_github("UPSUTER/GEMLI", subdir="GEMLI_package_v0"))

## Examples

```
m <- rowMeans(counts); cv2 <- apply(counts,1,var)/m^2; resid <- resid(lm(cv2 ~ m)); sel <- which(qm>=98 | (qm>=90 & qv>=40) | (qm>=60 & qv>=90))
```

## Evaluation signals

- The residual of the CV2-on-mean regression should be uncorrelated with mean expression, confirming variability is properly mean-corrected before quantile conversion
- The set exported to selected_memory_genes.csv must satisfy the exact quantile rule: mean quantile >= 98, or (mean >= 90 AND variability >= 40), or (mean >= 60 AND variability >= 90)
- The selected gene set should show clear enrichment of the Fig. 2b memory genes compared to a random gene subset of equal size
- Matrix orientation must be genes x rows, cells x columns; row means should have length equal to the number of genes
- Each selected gene's mean quantile and variability quantile values should fall within [0, 100] after conversion

## Limitations

- The selection criterion identifies candidate memory-gene populations and must be validated against the paper's Fig. 2b memory genes; it does not itself prove gene-expression memory
- GEMLI performs less robustly in cell populations containing only distantly related cells, which weakens the downstream lineage signal the gene selection feeds into
- Large variability in growth rates between lineages may decrease GEMLI's performance (cannot be excluded)
- GEMLI's scope is limited to lineages over about 1-5 cell divisions; long-term memory effects over weeks or months require other approaches

## Evidence

- [intro] intro gene stability: "We leverage genes with particularly stable expression over time to develop Gene Expression Memory-based Lineage Inference or GEMLI"
- [readme] readme package description: "GEMLI is an R package to predict cell lineages (cells with a common ancestor) from single cell RNA sequencing datasets and to call genes with a high gene expression memory"
- [readme] readme installation: "Simply run `library(devtools)` and then `install_github("UPSUTER/GEMLI", subdir="GEMLI_package_v0")`"
- [discussion] discussion robustness caveat: "However, GEMLI will generally perform less robustly in cell populations that contain only distantly related cells."
