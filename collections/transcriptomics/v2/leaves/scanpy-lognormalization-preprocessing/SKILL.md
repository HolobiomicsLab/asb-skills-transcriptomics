---
name: scanpy-lognormalization-preprocessing
description: Use when raw UMI count matrices (AnnData) from scRNA-seq datasets need
  normalization to comparable expression scales prior to cell-type scoring, marker
  identification, or label transfer.
when_to_use_negative:
- Input data is already normalized or provided as log(CPM + 1) or other normalized
  expression values rather than raw counts
- The downstream tool requires a different normalization convention, e.g. scNym which
  expects log(CPM + 1) with natural logarithm
- The analysis requires highly variable gene selection before integration — MASI deliberately
  skips HVG selection because only marker genes are used
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: Scanpy
  role: Performs the LogNormalization preprocessing (divide by total counts per cell,
    scale factor 10,000, log-transform) on raw count AnnData
  repo: https://github.com/hayatlab/MASI
- name: MASI
  role: Consumes the normalized expression matrix downstream for gene-to-cell-type
    score conversion and label transfer
  repo: https://github.com/hayatlab/MASI
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s42003-023-04820-3
    title: MASI enables fast model-free standardization and integration of single-cell
      transcriptomics data
  source_content_hash: sha256:fb22d433c8766bd9bd73ad6cacbf43f467b93fdf89227f5e39cab723f6397a9c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hayatlab/MASI
  tools:
  - MACA
  - MASI
  - Scanpy
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# scanpy-lognormalization-preprocessing

## Summary

Preprocess raw single-cell RNA-seq counts with LogNormalization — divide each cell's counts by its total, scale by 10,000, and log-transform — using Scanpy before marker-based integration with MASI.

## When to use

Use when raw UMI count matrices (AnnData) from scRNA-seq datasets need normalization to comparable expression scales prior to cell-type scoring, marker identification, or label transfer.

## When NOT to use

- Input data is already normalized or provided as log(CPM + 1) or other normalized expression values rather than raw counts
- The downstream tool requires a different normalization convention, e.g. scNym which expects log(CPM + 1) with natural logarithm
- The analysis requires highly variable gene selection before integration — MASI deliberately skips HVG selection because only marker genes are used

## Inputs

- Raw gene expression count matrix in Scanpy AnnData format (e.g. processed Wagner et al. or Farrell et al. zebrafish embryo datasets from the MASI figshare deposit 18866264)

## Outputs

- LogNormalized AnnData object (counts-per-10,000, log-transformed) ready for cell-type score conversion via masi.gene2cell

## How to apply

Load raw gene expression counts into a Scanpy AnnData object and apply LogNormalization: divide each gene's count by the total counts in that cell, multiply by a scale factor of 10,000, then log-transform. This normalization is a prerequisite step in the MASI pipeline — after normalization, the gene expression matrix is converted to a cell-type score matrix via masi.gene2cell with linearly weighted markers (1st marker 100%, Nth marker 50%) and a 0.25 percentile PlinerScore threshold. Notably, MASI skips highly variable gene selection after normalization because only identified marker genes are used for integrative annotation. Downstream, the normalized and scored data are annotated with masi.parallel (MACA over-clustering at resolutions 3, 5, 7 and neighborhood sizes 5, 10, 15) to ensemble labels across lineages.

## Related tools

- **Scanpy** (Performs the LogNormalization preprocessing (divide by total counts per cell, scale factor 10,000, log-transform) on raw count AnnData) — https://github.com/hayatlab/MASI
- **MASI** (Consumes the normalized expression matrix downstream for gene-to-cell-type score conversion and label transfer) — https://github.com/hayatlab/MASI

## Examples

```
import scanpy as sc; sc.pp.normalize_total(ad, target_sum=1e4); sc.pp.log1p(ad)
```

## Evaluation signals

- Each cell's post-normalization expression values sum to the scale factor (10,000) before log-transformation
- Values in the normalized matrix are non-negative and log-transformed (no raw integer counts remain)
- Downstream masi.gene2cell produces a cell-type score matrix and labels that feed masi.parallel with batch_size=50000 without errors
- For the zebrafish validation task, the integrated lineage map retrieves germline development back to at least the 3 hpf time point and lineage-timing calls match Fig. 1b–d (germline and epidermis at 3 hpf, notochord ~7 hpf, non-neural ectoderm separation at 12 hpf)

## Limitations

- The exact Scanpy version and any scanpy.pp.normalize_total parameter variations are not reported in the article (no software version numbers reported for Scanpy or other tools)
- LogNormalization parameters described in the discussion state 'X-percentile' and a default PlinerScore of 0.25, but the applied percentile value per analysis is not fully disambiguated

## Evidence

- [discussion] LogNormalization of raw counts (scale factor 10,000 followed by log-transformation): "Raw gene expression counts data were ‘LogNormalized’, which divides the total count in that cell and multiplies it by a scale factor of 10,000"
- [discussion] Skipping highly variable gene selection for MASI: "we skipped the step of calling highly variable genes, because only the identified marker genes were used for integrative annotation."
- [methods] gene2cell downstream use of normalized data: "scores, labels = masi.gene2cell(ad=ad,cell_markers=cell_markers,use_weight=True)"
- [other] Preprocessing step in zebrafish lineage workflow: [evidence span withheld — anchor status: not_found]
