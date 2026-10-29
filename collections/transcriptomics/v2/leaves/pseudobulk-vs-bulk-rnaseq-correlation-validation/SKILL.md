---
name: pseudobulk-vs-bulk-rnaseq-correlation-validation
description: Use when you need QC of a curated scRNA-seq time-course against a matched
  bulk RNA-seq reference — e.g. testing pseudo-bulk profiles of embryo stages against
  low-input bulk profiles before downstream analysis.
when_to_use_negative:
- The input is already a validated bulk-only expression matrix with no single-cell
  data to aggregate into pseudo-bulk profiles
- There is no matched bulk RNA-seq reference for the same stages, so agreement cannot
  be assessed
- The goal is cell-type-resolved communication inference (e.g. CellChat/CellPhoneDB)
  rather than dataset-level QC
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: Seurat
  role: Builds Seurat objects with cell filtering (CreateSeuratObject, min.features
    = 2000, min.cells = 0) on the scRNA-seq count matrices; v3.2.3 was used downstream
  repo: https://github.com/ChengLiLab/TimeTalk
- name: preprocessCore
  role: Quantile-normalizes the merged pseudo-bulk/bulk expression matrix via normalize.quantiles
    (version 1.48.0)
- name: TimeTalk
  role: Provides the custom R scripts for computing pseudo-bulk expression per stage
    and the curated low-input bulk RNA-seq reference data via the Figshare deposit
  repo: https://figshare.com/projects/TimeTalk_CB_manuscript_code_data/174498
- name: kBET
  role: Tests whether data points from different batches are well-mixed within random
    neighborhoods of a fixed size in PCA space
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s42003-023-05283-2
    title: TimeTalk uses single-cell RNA-seq datasets to decipher cell-cell communication
      during early embryo development
  source_content_hash: sha256:a507aa643327e1b791de87536d64c632bb0cb8ac5948a2680fe1761f1794235a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ChengLiLab/TimeTalk
  tools:
  - R
  - Seurat
  - TimeTalk
  - kBET
  - preprocessCore
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# pseudobulk-vs-bulk-rnaseq-correlation-validation

## Summary

Validates curated single-cell RNA-seq data quality by computing stage-level pseudo-bulk expression profiles and checking agreement with matching low-input bulk RNA-seq profiles via Pearson correlation, complemented by kBET batch-mixing assessment. Used in TimeTalk to confirm scRNA-seq (GSE38495, GSE45719) faithfully represents embryo development stages.

## When to use

Use when you need QC of a curated scRNA-seq time-course against a matched bulk RNA-seq reference — e.g. testing pseudo-bulk profiles of embryo stages against low-input bulk profiles before downstream analysis.

## When NOT to use

- The input is already a validated bulk-only expression matrix with no single-cell data to aggregate into pseudo-bulk profiles
- There is no matched bulk RNA-seq reference for the same stages, so agreement cannot be assessed
- The goal is cell-type-resolved communication inference (e.g. CellChat/CellPhoneDB) rather than dataset-level QC

## Inputs

- GEO scRNA-seq count matrices: GSE38495 (MII oocyte) and GSE45719 (E1-E4.0 mouse pre-implantation stages)
- Low-input bulk RNA-seq profiles from the TimeTalk Figshare deposit (figshare.com/projects/TimeTalk_CB_manuscript_code_data/174498)
- Custom R pseudo-bulk scripts from the TimeTalk Figshare project deposit

## Outputs

- Stage-wise pseudo-bulk expression matrix (average single cell per pre-implantation stage)
- Pearson correlation coefficients between pseudo-bulk and matching bulk profiles per stage (expect ~0.91 at late 2-cell)
- kBET batch-mixing statistics on PCA-space neighborhoods
- QC plots reproducing Fig. 1a-b and Supplementary Figs. 1-3

## How to apply

Download the GSE38495 (MII-oocyte) and GSE45719 (E1-E4.0) scRNA-seq count matrices from GEO plus the low-input bulk RNA-seq files from the TimeTalk Figshare deposit. Build Seurat objects with CreateSeuratObject(min.features = 2000, min.cells = 0) and use the custom R scripts shipped with the TimeTalk deposit to compute pseudo-bulk expression for an average single cell in each pre-implantation stage. Quantile-normalize the merged expression matrix with preprocessCore::normalize.quantiles (v1.48.0), then compute Pearson correlations between each stage's pseudo-bulk profile and its matching bulk profile — high agreement is expected (e.g. 0.91 at the late 2-cell stage). Run kBET on the PCA-space representation with a fixed neighborhood size to test whether batches are well-mixed within random neighborhoods, and plot pseudo-bulk-vs-bulk agreement and kBET results to reproduce Fig. 1a-b and Supplementary Figs. 1-3.

## Related tools

- **Seurat** (Builds Seurat objects with cell filtering (CreateSeuratObject, min.features = 2000, min.cells = 0) on the scRNA-seq count matrices; v3.2.3 was used downstream) — https://github.com/ChengLiLab/TimeTalk
- **preprocessCore** (Quantile-normalizes the merged pseudo-bulk/bulk expression matrix via normalize.quantiles (version 1.48.0))
- **TimeTalk** (Provides the custom R scripts for computing pseudo-bulk expression per stage and the curated low-input bulk RNA-seq reference data via the Figshare deposit) — https://figshare.com/projects/TimeTalk_CB_manuscript_code_data/174498
- **kBET** (Tests whether data points from different batches are well-mixed within random neighborhoods of a fixed size in PCA space)

## Examples

```
seu <- CreateSeuratObject(counts, min.features = 2000, min.cells = 0); mat.norm <- preprocessCore::normalize.quantiles(pseudobulk.mat)
```

## Evaluation signals

- Pearson correlation between matching stage pseudo-bulk and bulk profiles is high — expect r = 0.91 at the late 2-cell stage
- kBET reports good mixing of batches within random neighborhoods of fixed size in PCA space
- Cell filtering retained only cells with >= 2000 detected features (min.features = 2000, min.cells = 0)
- Merged expression matrix is quantile-normalized before correlation (preprocessCore::normalize.quantiles)
- Resulting plots reproduce Fig. 1a-b and Supplementary Figs. 1-3

## Limitations

- Granger causality conclusions in the broader pipeline are sensitive to the interpolation parameter winsz (Supplementary Figs. 20 and 21)
- The autocrine assumption of TimeTalk precludes exploration of maternal-embryo interactions, so bulk-matched validation covers embryo scRNA-seq only
- Inferred relationships ultimately require wet-lab confirmation; in silico agreement metrics are a QC reference, not biological proof

## Evidence

- [discussion] pseudo-bulk scRNA-seq and bulk agreement: "Custom R scripts were used for the pre-implantation scRNA-seq data to calculate the pseudo-bulk expression for an average single cell in each stage."
- [discussion] quantile normalization step: "Quantile normalization was performed with the function “normalize.quantiles” from R package preprocessCore (version 1.48.0)."
- [discussion] kBET assessment: "The kBET analysis assessed whether the data points with different batches in the PCA space were well-mixed within random neighborhoods of a fixed size."
- [discussion] Seurat filtering parameters: "CreateSeuratObject” in Seurat (parameters: min.features = 2000, min.cells = 0)."
- [discussion] data availability / figshare deposit: "we have uploaded our code and extensive data to Figshare (https://figshare.com/projects/TimeTalk_CB_manuscript_code_data/174498)"
- [discussion] GEO datasets source: "scRNA-seq datasets were downloaded from the GEO database (see Availability of data and materials)."
