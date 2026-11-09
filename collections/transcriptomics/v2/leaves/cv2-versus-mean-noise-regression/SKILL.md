---
name: cv2-versus-mean-noise-regression
description: Use when you have a raw gene-by-cell count matrix and need an unsupervised,
  purely data-driven set of highly and variably expressed candidate genes (e.g., memory-gene
  candidates for lineage inference) without external annotations.
when_to_use_negative:
- Input is already a filtered/normalized feature set or an existing memory-gene list
  — this criterion is meant for raw count matrices
- Populations containing only distantly related cells, where GEMLI's downstream lineage
  inference (and hence this gene preselection's purpose) performs less robustly
- Long-term lineage tracing over weeks or months, which requires genetic-mark-based
  approaches rather than GEMLI's gene-expression-memory selection
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: GEMLI
  role: R package whose items list hosts the count matrix and whose gene-selection
    step applies the mean/CV2-quantile criterion to nominate memory genes
  repo: https://github.com/UPSUTER/GEMLI
- name: R
  role: Language in which per-gene mean, CV2, the CV2-on-mean regression residuals,
    and the quantile thresholds are computed
- name: devtools
  role: Used to install the GEMLI package from GitHub before running the selection
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-024-47158-y
    title: Gene-expression memory-based prediction of cell lineages from scRNA-seq
      datasets
  source_content_hash: sha256:a35d5e46dff8701825ec2cf4d7bd90fe802da93c2670e23565b16de374707b3f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/UPSUTER/GEMLI
  tools:
  - GEMLI
  - R
  - devtools
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# cv2-versus-mean-noise-regression

## Summary

A gene-selection criterion from the GEMLI workflow that ranks genes from a raw scRNA-seq count matrix by per-gene mean expression and mean-corrected squared coefficient of variation (CV2), then selects genes via quantile cutoffs. It was designed to enrich for 'memory genes' whose expression is stable across cell lineages.

## When to use

Use when you have a raw gene-by-cell count matrix and need an unsupervised, purely data-driven set of highly and variably expressed candidate genes (e.g., memory-gene candidates for lineage inference) without external annotations.

## When NOT to use

- Input is already a filtered/normalized feature set or an existing memory-gene list — this criterion is meant for raw count matrices
- Populations containing only distantly related cells, where GEMLI's downstream lineage inference (and hence this gene preselection's purpose) performs less robustly
- Long-term lineage tracing over weeks or months, which requires genetic-mark-based approaches rather than GEMLI's gene-expression-memory selection

## Inputs

- Raw gene-by-cell count matrix (genes as rows, cells as columns), e.g., loaded from GEMLI_example_data_matrix.RData
- GEMLI items list with a 'gene_expression' slot

## Outputs

- Selected gene identifiers (e.g., selected_memory_genes.csv) representing highly expressed and/or mean-corrected-highly-variable genes
- Per-gene mean expression and CV2-residual variability quantiles

## How to apply

Load the raw count matrix (e.g., GEMLI_example_data_matrix.RData) with genes as rows and cells as columns into the 'gene_expression' slot of a GEMLI items list. Compute per-gene mean expression and squared coefficient of variation (CV2), then regress CV2 on mean expression and take the regression residuals as the variability measure, so variability is residualized against the mean-driven noise floor. Convert mean expression and residual variability to quantiles across all genes, and apply the selection rule: keep genes with mean quantile >= 98, OR (mean quantile >= 90 AND variability quantile >= 40), OR (mean quantile >= 60 AND variability quantile >= 90). Export the retained gene identifiers (e.g., to selected_memory_genes.csv) and use them as the input gene universe for downstream GEMLI lineage prediction.

## Related tools

- **GEMLI** (R package whose items list hosts the count matrix and whose gene-selection step applies the mean/CV2-quantile criterion to nominate memory genes) — https://github.com/UPSUTER/GEMLI
- **R** (Language in which per-gene mean, CV2, the CV2-on-mean regression residuals, and the quantile thresholds are computed)
- **devtools** (Used to install the GEMLI package from GitHub before running the selection)

## Examples

```
install_github("UPSUTER/GEMLI", subdir="GEMLI_package_v0"); library(GEMLI); m <- mean(expr); cv2 <- apply(expr,1,var)/m^2; resid <- lm(cv2~m)$resid; sel <- which(mq>=98 | (mq>=90 & vq>=40) | (mq>=60 & vq>=90))
```

## Evaluation signals

- Genes with mean-expression quantile >= 98 are always retained; no gene below mean quantile 60 is retained
- Every retained gene satisfies at least one of the three quantile rules (98; 90/40; 60/90)
- The CV2 measure used for variability is the regression residual of CV2 on mean expression, not raw CV2
- The selected gene set is enriched for the memory genes highlighted in Fig. 2b of the GEMLI paper, rather than resembling a random gene subset
- Output gene identifiers match the row names of the input count matrix

## Limitations

- Selection enriches for memory genes but is a heuristic quantile rule; it does not directly test memory and may include non-memory genes
- GEMLI performs less robustly in cell populations that contain only distantly related cells
- Large variability in growth rates between lineages may decrease downstream GEMLI performance
- Scope is limited to lineages over about 1-5 cell divisions; longer time spans require other approaches

## Evidence

- [methods] GEMLI selects genes based on high mean expression and high mean-corrected CV2 variability (variability residualized against mean), and this selection highly enriches for memory genes.: [evidence span withheld — anchor status: not_found]
- [methods] keep genes with mean quantile >= 98, or (mean quantile >= 90 AND variability quantile >= 40), or (mean quantile >= 60 AND variability quantile >= 90): [evidence span withheld — anchor status: not_found]
- [readme] The approach is based on findings of Phillips et al. 2019 (doi.org/10.1038/s41467-019-09189-8) where it was shown that some genes show varying gene expression across cell lineages that is stable over multiple cell generations.: "The approach is based on findings of Phillips et al. 2019 (doi.org/10.1038/s41467-019-09189-8) where it was shown that some genes show varying gene expression across cell lineages"
- [readme] Simply run `library(devtools)` and then `install_github("UPSUTER/GEMLI", subdir="GEMLI_package_v0")`: "Simply run `library(devtools)` and then `install_github("UPSUTER/GEMLI", subdir="GEMLI_package_v0")`."
