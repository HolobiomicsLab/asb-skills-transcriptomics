---
name: public-geo-dataset-retrieval
description: Use when a benchmark requires 25 real scRNA-seq datasets with known cell-type
  labels and accession IDs (GSE/E-MTAB/SRP), or when populating the scISR ARI comparison
  pipeline with raw expression matrices from GEO/ArrayExpress.
when_to_use_negative:
- The task involves the 116 simulated datasets rather than real scRNA-seq data — simulations
  use the paper's synthetic generation procedure (zeroing 30% of values in 40% of
  genes), not GEO retrieval
- The dataset is proprietary, restricted-access, or lacks known cell-type labels,
  since ARI evaluation against ground-truth labels is required
- Data is already imputed or filtered with different criteria (e.g. genes retained
  despite <1% non-zero entries), which would break comparability with the reported
  mean ARIs
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scISR
  role: Downstream consumer of the retrieved GEO/ArrayExpress datasets; ships a bundled
    example dataset (Goolam) and performs hypothesis-testing plus subspace-regression
    imputation on the downloaded matrices
  repo: https://github.com/duct317/scISR
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-022-06500-4
    title: A novel method for single-cell data imputation using subspace regression
  source_content_hash: sha256:04b0a4cb564ca1a4ae0710f9ba9bc07a8b5f24689e6e0a4114f4e2fcbea8864a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/duct317/scISR
  tools:
  - MAGIC
  - SAVER
  - scISR
  - scImpute
  - scScope
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# public-geo-dataset-retrieval

## Summary

Retrieves publicly deposited scRNA-seq datasets (e.g. GSE84133, GSE75140, GSE45719, E-MTAB-3321, GSE65525) from GEO/ArrayExpress to build benchmark inputs for the scISR-vs-competitor ARI evaluation, with standard gene filtering and log2 rescaling applied before imputation.

## When to use

Use when a benchmark requires 25 real scRNA-seq datasets with known cell-type labels and accession IDs (GSE/E-MTAB/SRP), or when populating the scISR ARI comparison pipeline with raw expression matrices from GEO/ArrayExpress.

## When NOT to use

- The task involves the 116 simulated datasets rather than real scRNA-seq data — simulations use the paper's synthetic generation procedure (zeroing 30% of values in 40% of genes), not GEO retrieval
- The dataset is proprietary, restricted-access, or lacks known cell-type labels, since ARI evaluation against ground-truth labels is required
- Data is already imputed or filtered with different criteria (e.g. genes retained despite <1% non-zero entries), which would break comparability with the reported mean ARIs

## Inputs

- GEO/ArrayExpress accession IDs (GSE84133, GSE75140, GSE45719, E-MTAB-3321, GSE65525, GSE115746, GSE139598, GSE53386, GSE52583, GSE36552, SRP041736, GSE67835, GSE59739, ...)
- Publicly available scRNA-seq count or normalized expression matrices (FPKM, RPKM, TPM, CPM, RPM, UMI counts)
- Known cell-type label vectors for each dataset

## Outputs

- Preprocessed gene-by-cell expression matrices (genes expressed in <2 cells and genes with <1% non-zero entries removed; log2-rescaled when value range >100)
- Per-dataset cell-type label sets used to set k for k-means and compute ARI
- Per-dataset ARI values aggregated into the 25-dataset mean ARI table per imputation method

## How to apply

Resolve each accession ID (GSE, E-MTAB, SRP) from the article's benchmark table (Fan GSE53386, Treutlein GSE52583, Yan GSE36552, Goolam E-MTAB-3321, Deng GSE45719, Pollen SRP041736, Darmanis GSE67835, Usoskin GSE59739, Camp GSE75140, Klein GSE65525, etc.) and download raw count/FPKM/TPM/RPKM matrices plus cell-type labels from GEO or ArrayExpress. For datasets with a value range larger than 100, rescale using a base-2 log transformation. Preprocess by removing genes expressed in less than two cells and genes with less than one percent of non-zero-valued entries, which matches the paper's filtering criteria. Each dataset must carry known cell-type labels so downstream k-means clustering with the true number of cell types k can be scored by ARI. Rationale: the 25-dataset ARI benchmark (scISR 0.571 vs raw 0.504, MAGIC 0.461, scImpute 0.286, SAVER 0.423, scScope 0.165, scGNN 0.279) is only reproducible if the same public data, filters, and normalization are used.

## Related tools

- **scISR** (Downstream consumer of the retrieved GEO/ArrayExpress datasets; ships a bundled example dataset (Goolam) and performs hypothesis-testing plus subspace-regression imputation on the downloaded matrices) — https://github.com/duct317/scISR

## Examples

```
library(scISR); data('Goolam'); raw <- Goolam$data; label <- Goolam$label; imputed <- scISR(data = raw)
```

## Evaluation signals

- Retrieved dataset list matches the paper's benchmark accessions (GSE53386, GSE52583, GSE36552, E-MTAB-3321, GSE45719, SRP041736, GSE67835, GSE59739, GSE75140, GSE65525, GSE84133, GSE115746, GSE139598, ...)
- After filtering, genes expressed in fewer than two cells and genes with fewer than one percent non-zero entries are absent from every matrix
- Datasets whose value range exceeded 100 show a log2-transformed range, consistent with the rescaling rule
- Each dataset retains true cell-type labels so k-means with the true number of cell types k yields ARI against ground truth
- Reproduced mean ARIs per method match reported values (scISR 0.571, raw 0.504, MAGIC 0.461, scImpute 0.286, SAVER 0.423, scScope 0.165, scGNN 0.279) with one-sided Wilcoxon p-values 9.8e-6 (vs methods) and 3.2e-5 (vs raw)

## Limitations

- The 25 real scRNA-seq datasets are not exhaustively named with accession IDs in the discussion text, so some accessions must be traced from the results table or supplementary material
- Dataset-specific quantification schemes differ (FPKM, RPKM, TPM, CPM, RPM, UMI counts), so cross-dataset comparability depends on applying the paper's normalization/filters per dataset
- No software version stated for scISR or the R stats package, and the GitHub repository has no changelog, limiting exact reproducibility of the retrieved-data pipeline

## Evidence

- [discussion] using 25 real scRNA-seq datasets and 116 simulated datasets.: "using 25 real scRNA-seq datasets and 116 simulated datasets."
- [results] For datasets with the range of values larger than 100, we rescale the data using log transformation (base 2).: "For datasets with the range of values larger than 100, we rescale the data using log transformation (base 2)."
- [results] (i) genes expressed in less than two cells; and (ii) genes that have less than one percent of non-zero-valued entries.: "(i) genes expressed in less than two cells; and (ii) genes that have less than one percent of non-zero-valued entries."
- [results] 1. Fan29GSE53386Mouse EmbryoSUPeR-seqPlateReadsFPKM0.584669: "1. Fan29GSE53386Mouse EmbryoSUPeR-seqPlateReadsFPKM0.584669"
- [readme] Load Goolam dataset: data('Goolam'); raw <- Goolam$data; label <- Goolam$label: [evidence span withheld — anchor status: external_doc]
