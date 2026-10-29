---
name: scanpy-normalization-qc-preprocessing
description: Use when raw count matrices (e.g., CSV count tables for SC or ST data)
  must be normalized, log-transformed, and QC-metric'd before feeding a mapping/integration
  model, or when a median-genes dropout rate is needed.
when_to_use_negative:
- Input is already normalized/log-transformed (e.g., a processed h5ad or the Figshare-deposited
  preprocessed matrices) — re-normalizing would double-scale the data.
- Data are not raw counts (e.g., TPM/CPM or integer-free matrices), where sc.pp.normalize_total's
  library-size assumption no longer applies.
- Spatial coordinates or cell-type labels are the analysis target — this skill only
  transforms expression matrices and computes QC metrics.
edam_operation: http://edamontology.org/operation_3562
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scanpy
  role: Performs normalize_total, log1p, and calculate_qc_metrics on AnnData objects
    built from the SC and ST count CSVs
  repo: https://github.com/WhirlFirst/STEM
- name: STEM (scSTEM)
  role: Consumes the normalized/log-scaled SC and ST matrices; its dp parameter is
    set from the median n_genes_by_counts ratio computed here
  repo: https://github.com/WhirlFirst/STEM
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s42003-023-05640-1
    title: STEM enables mapping of single-cell and spatial transcriptomics data with
      transfer learning
  source_content_hash: sha256:00cca3c0402246cc9f2bffd25ed15f25db120cfa365bd33a040172ae27e4ff0d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WhirlFirst/STEM
  tools:
  - CellTrek
  - STEM
  - STEM (scSTEM)
  - Seurat
  - Spaotsc
  - Tangram
  - scSpace
  - scanpy
  edam_operation: http://edamontology.org/operation_3562
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# scanpy-normalization-qc-preprocessing

## Summary

Normalize total counts, log1p-transform, and compute QC metrics (n_genes_by_counts) on single-cell and spatial transcriptomics count matrices with scanpy before model training; used in the STEM pipeline to prepare E1z2 SC/ST data and derive the dropout rate.

## When to use

Use when raw count matrices (e.g., CSV count tables for SC or ST data) must be normalized, log-transformed, and QC-metric'd before feeding a mapping/integration model, or when a median-genes dropout rate is needed.

## When NOT to use

- Input is already normalized/log-transformed (e.g., a processed h5ad or the Figshare-deposited preprocessed matrices) — re-normalizing would double-scale the data.
- Data are not raw counts (e.g., TPM/CPM or integer-free matrices), where sc.pp.normalize_total's library-size assumption no longer applies.
- Spatial coordinates or cell-type labels are the analysis target — this skill only transforms expression matrices and computes QC metrics.

## Inputs

- Raw gene-by-cell count matrix as CSV (e.g., simu_sc_counts.csv, simu_st_counts.csv from the E1z2 SeqFISH mouse data)
- Cell/spot metadata CSV (e.g., metadata.csv with x_global/y_global, simu_st_metadata.csv with spatial coordinates)

## Outputs

- Normalized, log1p-transformed expression DataFrames for SC and ST data
- AnnData objects with obsm['spatial'] populated from spatial metadata
- Per-cell/spot n_genes_by_counts QC metric and the derived dropout rate dp (e.g., 0.5836734693877551 for E1z2)

## How to apply

Load the raw count tables (e.g., simu_sc_counts.csv transposed so cells are rows, simu_st_counts.csv) into AnnData objects with metadata in obs. Apply sc.pp.normalize_total followed by sc.pp.log1p to each AnnData independently, converting the resulting X back into a pandas DataFrame. Then run sc.pp.calculate_qc_metrics(adata, percent_top=None, log1p=False, inplace=True) on both objects and read the median of adata.obs['n_genes_by_counts']; the dropout rate for STEM is dp = 1 - median(SC n_genes_by_counts)/median(ST n_genes_by_counts), which equals ~0.584 for the E1z2 mouse SeqFISH data. The rationale is that SC and ST data have different depth/dropout characteristics, and both the shared embedding training and the dropout-regularized model require comparably scaled, log-transformed inputs.

## Related tools

- **scanpy** (Performs normalize_total, log1p, and calculate_qc_metrics on AnnData objects built from the SC and ST count CSVs) — https://github.com/WhirlFirst/STEM
- **STEM (scSTEM)** (Consumes the normalized/log-scaled SC and ST matrices; its dp parameter is set from the median n_genes_by_counts ratio computed here) — https://github.com/WhirlFirst/STEM

## Examples

```
sc.pp.normalize_total(adata); sc.pp.log1p(adata); sc.pp.calculate_qc_metrics(adata, percent_top=None, log1p=False, inplace=True); dp = 1 - adata.obs['n_genes_by_counts'].median()/stadata.obs['n_genes_by_counts'].median()
```

## Evaluation signals

- Both AnnData objects pass through sc.pp.normalize_total then sc.pp.log1p in that order, with values non-negative and no library-size dependence remaining in row sums.
- sc.pp.calculate_qc_metrics is called with percent_top=None and log1p=False so n_genes_by_counts reflects the raw gene-detection count, not a logged value.
- The dropout rate equals 1 - median(SC n_genes_by_counts)/median(ST n_genes_by_counts); for E1z2 it should reproduce ~0.5836734693877551.
- Output DataFrames preserve the original obs_names/var_names of the input CSVs (cells as rows after transposing scdata).
- The resulting scdata/stdata matrices are float tensors that train without NaN/loss divergence over 400 STEM epochs.

## Limitations

- The dropout-rate heuristic assumes the median-genes ratio between SC and ST data is a meaningful proxy for technical dropout; it was tuned on SeqFISH E1z2 data and may not transfer to other platforms.
- No additional filtering (e.g., gene/cell QC cutoffs beyond QC-metric computation) is performed here; downstream quality depends entirely on the raw counts supplied.

## Evidence

- [readme] Then we normalized and log-scaled these data.: "First, we load the simulated ST data as ST data and the raw SeqFISH data as SC data. Then we normalized and log-scaled these data."
- [readme] dropout rate computed from median genes-by-counts ratio: "dp = 1- adata.obs['n_genes_by_counts'].median()/stadata.obs['n_genes_by_counts'].median() #0.5836734693877551"
- [methods] filter steps from article methods: "Next we calculate the ratio between the median total counts value of SC and ST data as the dropout rate."
- [readme] normalize then log1p ordering: "sc.pp.normalize_total(adata)
sc.pp.log1p(adata)"
