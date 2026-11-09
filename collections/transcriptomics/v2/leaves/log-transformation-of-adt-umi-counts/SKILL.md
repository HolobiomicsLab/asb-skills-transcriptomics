---
name: log-transformation-of-adt-umi-counts
description: Use when raw ADT/UMI count matrices (cell-containing and/or empty droplets)
  from CITE-seq, ASAP-seq, or TEA-seq data must be transformed before estimating per-antibody
  ambient background or before computing ambient correlation summaries.
when_to_use_negative:
- Input is already dsb-normalized (DSBNormalizeProtein output) — re-log-transforming
  normalized residuals would distort the standardized values
- Data are RNA expression counts rather than ADT protein counts — RNA counts are log-normalized
  by different workflows (e.g., Seurat/Scanpy conventions), not the dsb ADT transform
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: dsb
  role: R package providing the raw CITE-seq example matrices and the normalization
    pipeline into which the log-transformed ADT counts are fed
  repo: https://cran.r-project.org/package=dsb
- name: R
  role: Language in which the log transformation and per-protein mean log10(count
    + 1) summaries are computed (base R log/log10 and cor)
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-022-29356-8
    title: Normalizing and denoising protein expression data from droplet-based single
      cell profiling
  source_content_hash: sha256:2d4aa59844b9994d1dffcf48ac86f371dfb655ad1104fad7a414bbdc8e86529d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - dsb
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# log-transformation-of-adt-umi-counts

## Summary

Log-transform raw antibody-derived tag (ADT) UMI count matrices from droplet-based single cell experiments (CITE-seq and related assays) prior to ambient background estimation and dsb normalization. The transform stabilizes the wide dynamic range of ADT counts so that ambient noise levels in empty droplets can be compared with cell background levels.

## When to use

Use when raw ADT/UMI count matrices (cell-containing and/or empty droplets) from CITE-seq, ASAP-seq, or TEA-seq data must be transformed before estimating per-antibody ambient background or before computing ambient correlation summaries.

## When NOT to use

- Input is already dsb-normalized (DSBNormalizeProtein output) — re-log-transforming normalized residuals would distort the standardized values
- Data are RNA expression counts rather than ADT protein counts — RNA counts are log-normalized by different workflows (e.g., Seurat/Scanpy conventions), not the dsb ADT transform

## Inputs

- Raw ADT/protein UMI count matrix for cell-containing droplets (e.g., dsb::raw_pbmc_mtx_example_87ab_citeseq_Tsang, dsb::cells_citeseq_mtx)
- Raw ADT/protein UMI count matrix for empty droplets (e.g., dsb::raw_emptydrop_mtx_example_87ab_citeseq_Tsang, dsb::empty_drop_citeseq_mtx)

## Outputs

- Log-transformed ADT matrix (e.g., dlog = log(cell + 10)) suitable for dsb step I/II normalization
- Per-protein mean log10(count + 1) vectors for empty droplets and cells used in ambient background correlation tables (ambient_correlation_results.csv)

## How to apply

Apply an element-wise log transformation to the raw ADT UMI count matrix; the dsb methods use log(cell + 10) on the cell protein matrix, while per-protein background summaries in the ambient-correlation workflow use mean log10(count + 1) over all barcodes. Compute the transformed values separately for the cell-containing droplet matrix and the empty-droplet matrix loaded from the dsb package (e.g., dsb::raw_pbmc_mtx_example_87ab_citeseq_Tsang and dsb::raw_emptydrop_mtx_example_87ab_citeseq_Tsang). Rationale: ADT reads from empty droplets can exceed cell-containing droplets more than tenfold, so the log scale is required before per-antibody background means (mu_N) are subtracted and standardized by the background standard deviation (sigma_N) in dsb step I, and before the two-component Gaussian mixture model is fit per cell in step II. Decide the offset based on downstream use: the dsb code uses +10 in dlog = log(cell + 10), while background-correlation tables use log10(count + 1) per protein.

## Related tools

- **dsb** (R package providing the raw CITE-seq example matrices and the normalization pipeline into which the log-transformed ADT counts are fed) — https://cran.r-project.org/package=dsb
- **R** (Language in which the log transformation and per-protein mean log10(count + 1) summaries are computed (base R log/log10 and cor))

## Examples

```
dlog = log(cell + 10)
```

## Evaluation signals

- Transformed matrix should contain no negative values and preserve the original dimensions (all barcodes × 87 ADT proteins for the 87-antibody PBMC example)
- Per-protein mean log10(count + 1) computed over empty droplets vs. cells should yield a strong positive Pearson correlation, consistent with Fig. 1b ('Background levels in the empty droplets and cells were highly correlated')
- The ambient-correlation output table should contain exactly one row per protein (87 rows) with columns for protein name, empty-droplet mean log10(count + 1), and cell mean log10(count + 1)
- After subsequent dsb step I standardization, values should be expressed as (x − mu_N)/sigma_N, i.e., roughly zero-centered per antibody

## Limitations

- Defining a reasonable subset of barcodes as background droplets still requires care; the transform itself does not remove sensitivity of dsb results to background-droplet definition
- Proteins with very low raw signal (maximum UMI count < 5 across all cells, e.g., CD34 removed in two datasets) should be removed prior to normalization, so the log transform should be applied after such filtering
- The exact offset differs across contexts (log(cell + 10) in dsb methods code vs. log10(count + 1) in background summaries); mixing offsets inconsistently will change background means and correlations

## Evidence

- [methods] log transformation of ADT counts: "# log transformation
dlog = log(cell + 10)"
- [methods] standardization by background mean and standard deviation: "Subtract background mean $\mu_{N}$ and divide by the background standard deviation $\sigma_{N}$"
- [discussion] ambient background estimated from empty droplets: "In step I, protein counts in empty droplets are used to estimate the expected ambient background noise for each antibody"
- [readme] empty droplet reads closely match cell levels: [evidence span withheld — anchor status: external_doc]
