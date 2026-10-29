---
name: quantile-based-gene-filtering
description: Use when you have a raw gene-by-cell count matrix (e.g. from GEMLI_example_data_matrix.RData)
  and need to pre-select a small set of highly expressed and variably expressed candidate
  memory genes before lineage inference.
when_to_use_negative:
- Input is normalized/transformed expression (e.g. TPM or log-normalized data) rather
  than raw counts — the CV2-vs-mean regression and quantile thresholds are calibrated
  on raw count dispersion
- The goal is generic highly-variable-gene selection for clustering/dimensionality
  reduction; this rule is tuned to enrich for memory genes used by GEMLI lineage prediction,
  not for exploratory HVG workflows
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2269
tools:
- name: GEMLI
  role: R package whose gene-selection step this skill reproduces; the count matrix
    is placed in the 'gene_expression' slot of a GEMLI items list
  repo: https://github.com/UPSUTER/GEMLI
- name: R
  role: Language used to compute mean/CV2, run the CV2-on-mean regression, and apply
    the quantile thresholds
  repo: https://github.com/UPSUTER/GEMLI
- name: devtools
  role: Used to install the GEMLI package from GitHub via install_github
  repo: https://github.com/UPSUTER/GEMLI
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-024-47158-y
    title: Gene-expression memory-based prediction of cell lineages from scRNA-seq
      datasets
  source_content_hash: sha256:4524976128ef5960a64f306b92d829df459a90169e2ff77a680fe1c905255186
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/UPSUTER/GEMLI
  tools:
  - GEMLI
  - R
  - devtools
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# quantile-based-gene-filtering

## Summary

Reproduces GEMLI's initial memory-gene selection criterion: from a raw scRNA-seq count matrix, per-gene mean expression and mean-corrected squared coefficient of variation (CV2) are converted to quantiles, and a compound quantile rule selects genes that are highly enriched for gene-expression-memory genes.

## When to use

Use when you have a raw gene-by-cell count matrix (e.g. from GEMLI_example_data_matrix.RData) and need to pre-select a small set of highly expressed and variably expressed candidate memory genes before lineage inference.

## When NOT to use

- Input is normalized/transformed expression (e.g. TPM or log-normalized data) rather than raw counts — the CV2-vs-mean regression and quantile thresholds are calibrated on raw count dispersion
- The goal is generic highly-variable-gene selection for clustering/dimensionality reduction; this rule is tuned to enrich for memory genes used by GEMLI lineage prediction, not for exploratory HVG workflows

## Inputs

- Raw gene expression count matrix (rows = genes, columns = cells), e.g. loaded from GEMLI_example_data_matrix.RData into the 'gene_expression' slot of a GEMLI items list
- Optional: list of reference memory genes from Fig. 2b of the paper for enrichment validation

## Outputs

- Vector of selected gene identifiers passing the compound quantile rule
- selected_memory_genes.csv containing the selected memory gene set
- Quantile-transformed per-gene mean expression and mean-corrected CV2 variability tables (intermediate)

## How to apply

Load the raw count matrix into the 'gene_expression' slot of a GEMLI items list with genes as rows and cells as columns. Compute per-gene mean expression and squared coefficient of variation (CV2), then linearly regress CV2 on mean expression and take the regression residuals as the mean-corrected variability measure — this prevents simply re-selecting highly expressed genes. Convert both mean expression and residual variability to quantiles across all genes and keep genes satisfying: mean quantile >= 98, OR (mean quantile >= 90 AND variability quantile >= 40), OR (mean quantile >= 60 AND variability quantile >= 90). Export the surviving gene identifiers to a file such as selected_memory_genes.csv. The rationale, per the paper, is that genes with particularly stable expression over time underpin GEMLI lineage prediction, and this quantile rule recovers the memory-gene population highlighted in Fig. 2b rather than a random gene subset.

## Related tools

- **GEMLI** (R package whose gene-selection step this skill reproduces; the count matrix is placed in the 'gene_expression' slot of a GEMLI items list) — https://github.com/UPSUTER/GEMLI
- **R** (Language used to compute mean/CV2, run the CV2-on-mean regression, and apply the quantile thresholds) — https://github.com/UPSUTER/GEMLI
- **devtools** (Used to install the GEMLI package from GitHub via install_github) — https://github.com/UPSUTER/GEMLI

## Examples

```
library(devtools); install_github("UPSUTER/GEMLI", subdir="GEMLI_package_v0"); library(GEMLI)
```

## Evaluation signals

- Genes with mean-expression quantile >= 98 must all be retained by the rule, independent of variability
- The residual variability measure should be approximately uncorrelated with mean expression (mean-corrected CV2), confirming the regression removed the mean dependency
- The selected gene set shows clear enrichment of the memory genes highlighted in Fig. 2b of the paper compared to random gene subsets of equal size
- Selected identifiers export cleanly to selected_memory_genes.csv with gene names matching row names of the input count matrix
- Rule branches (mean>=90 & var>=40; mean>=60 & var>=90) contribute additional genes beyond the mean-quantile>=98 set

## Limitations

- The selection is a pre-filter for GEMLI's memory-based lineage inference; GEMLI performs less robustly in cell populations that contain only distantly related cells, so this filter alone does not guarantee lineage signal
- The quantile cutoffs (98/90/40/60/90) were calibrated on the GEMLI example/paper datasets and may need re-validation on datasets with very different dispersion profiles
- Applies to small to medium-sized lineages over roughly 1-5 cell divisions; longer-term lineage signals may be outside GEMLI's scope

## Evidence

- [intro] We leverage genes with particularly stable expression over time to develop Gene Expression Memory-based Lineage Inference or GEMLI: "We leverage genes with particularly stable expression over time to develop Gene Expression Memory-based Lineage Inference or GEMLI"
- [results] GEMLI selects genes based on high mean expression and high mean-corrected CV2 variability: [evidence span withheld — anchor status: not_found]
- [methods] install_github: "Simply run `library(devtools)` and then `install_github("UPSUTER/GEMLI", subdir="GEMLI_package_v0")`."
- [readme] README memory genes statement: "The approach is based on findings of Phillips et al. 2019 (doi.org/10.1038/s41467-019-09189-8) where it was shown that some genes show varying gene expression across cell lineages"
