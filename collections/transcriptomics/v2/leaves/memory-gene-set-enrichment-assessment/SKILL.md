---
name: memory-gene-set-enrichment-assessment
description: Use when a raw scRNA-seq count matrix must be reduced to the GEMLI memory-gene
  candidate set, or when testing whether a quantile-based mean/CV2 selection rule
  enriches for gene-expression-memory genes.
when_to_use_negative:
- Input is not a raw count matrix but already a filtered/normalized gene set or feature
  table — the quantile rule operates on the full mean/CV2 distribution across all
  genes
- The goal is lineage prediction itself (use GEMLI's predict_lineages on the prepared
  GEMLI_items instead) rather than the upstream memory-gene selection step
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: GEMLI
  role: R package whose example data (GEMLI_example_data_matrix.RData) and memory-gene
    selection logic are being reconstructed; installed from GitHub
  repo: https://github.com/UPSUTER/GEMLI
- name: R
  role: Environment in which mean/CV2 computation, CV2-on-mean regression, quantile
    calculation, and CSV export are performed
  repo: https://github.com/UPSUTER/GEMLI
- name: devtools
  role: Used to install the GEMLI package from GitHub before loading the example count
    matrix
  repo: https://github.com/UPSUTER/GEMLI
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-024-47158-y
    title: Gene-expression memory-based prediction of cell lineages from scRNA-seq
      datasets
  source_content_hash: sha256:ae1e1f5c7b3c7dd90b4b23e7f9cf81787cb529821cca43ea96e4e8b956f5ee86
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/UPSUTER/GEMLI
  tools:
  - GEMLI
  - R
  - devtools
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# memory-gene-set-enrichment-assessment

## Summary

Reconstructs GEMLI's initial memory-gene selection criterion: from a raw scRNA-seq count matrix (genes x cells), per-gene mean expression and mean-corrected CV2 variability are computed and combined into a quantile-based rule that yields a gene set highly enriched for the memory genes reported in Fig. 2b of Eisele et al. 2024.

## When to use

Use when a raw scRNA-seq count matrix must be reduced to the GEMLI memory-gene candidate set, or when testing whether a quantile-based mean/CV2 selection rule enriches for gene-expression-memory genes.

## When NOT to use

- Input is not a raw count matrix but already a filtered/normalized gene set or feature table — the quantile rule operates on the full mean/CV2 distribution across all genes
- The goal is lineage prediction itself (use GEMLI's predict_lineages on the prepared GEMLI_items instead) rather than the upstream memory-gene selection step

## Inputs

- Raw gene expression count matrix (rows = genes, columns = cells), e.g. from GEMLI_example_data_matrix.RData
- GEMLI items list with a populated 'gene_expression' slot (R)

## Outputs

- selected_memory_genes.csv containing the gene identifiers passing the quantile-based selection rule
- Per-gene mean expression and mean-corrected CV2 (regression residual) variability values

## How to apply

In R, load the raw gene expression matrix (e.g. GEMLI_example_data_matrix.RData from the UPSUTER/GEMLI repository, GEMLI_package_v0) and place it in the 'gene_expression' slot of a GEMLI items list with rows as genes and columns as cells. Compute per-gene mean expression and squared coefficient of variation (CV2), then linearly regress CV2 on mean expression and take the regression residuals as the variability measure, so that variability is corrected for its mean dependence. Convert mean expression and residual variability to quantiles across all genes and apply the selection rule: keep genes with mean quantile >= 98, or (mean quantile >= 90 AND variability quantile >= 40), or (mean quantile >= 60 AND variability quantile >= 90). Export the selected gene identifiers to selected_memory_genes.csv and validate by checking overlap with the memory genes highlighted in Fig. 2b of the paper; the rule should recover the reported memory-gene population rather than a random gene subset.

## Related tools

- **GEMLI** (R package whose example data (GEMLI_example_data_matrix.RData) and memory-gene selection logic are being reconstructed; installed from GitHub) — https://github.com/UPSUTER/GEMLI
- **R** (Environment in which mean/CV2 computation, CV2-on-mean regression, quantile calculation, and CSV export are performed) — https://github.com/UPSUTER/GEMLI
- **devtools** (Used to install the GEMLI package from GitHub before loading the example count matrix) — https://github.com/UPSUTER/GEMLI

## Examples

```
library(devtools); install_github("UPSUTER/GEMLI", subdir="GEMLI_package_v0"); library(GEMLI); load("GEMLI_example_data_matrix.RData")
```

## Evaluation signals

- Selected gene set overlaps the memory genes highlighted in Fig. 2b of Eisele et al. 2024, i.e. it is enriched for memory genes relative to a random gene subset of the same size
- Every selected gene satisfies at least one of the three quantile conditions: mean >= 98; (mean >= 90 AND CV2-residual variability >= 40); (mean >= 60 AND variability >= 90)
- Variability values are regression residuals (CV2 regressed on mean expression), not raw CV2, so mean-corrected values are used for the >= 40 and >= 90 quantile branches
- selected_memory_genes.csv contains gene identifiers matching the row names of the input count matrix

## Limitations

- The quantile thresholds (98/90/40/60/90) are dataset-specific and derived from the GEMLI example data; applying them to datasets with very different mean-variance structure may shift the selected set
- The mean-corrected variability depends on the CV2-on-mean linear regression; genes deviating from this mean-variance trend may be mis-ranked
- The memory-gene enrichment validation is anchored to Fig. 2b of this paper, so no external ground truth is available for other tissues or datasets

## Evidence

- [abstract] GEMLI memory-gene selection rationale: "Here we introduce Gene Expression Memory-based Lineage Inference (GEMLI), a computational tool allowing to robustly identify small to medium-sized cell lineages solely from scRNA-seq datasets."
- [readme] GEMLI package scope: "GEMLI is an R package to predict cell lineages (cells with a common ancestor) from single cell RNA sequencing datasets and to call genes with a high gene expression memory on the predicted cell"
- [methods] GEMLI installation: "Simply run `library(devtools)` and then `install_github("UPSUTER/GEMLI", subdir="GEMLI_package_v0")`."
