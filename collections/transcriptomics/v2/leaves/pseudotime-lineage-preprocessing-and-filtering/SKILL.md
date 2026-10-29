---
name: pseudotime-lineage-preprocessing-and-filtering
description: Use when preparing multi-sample scRNA-seq data (e.g., HCA bone marrow
  counts with cell-to-sample annotation and Slingshot/TSCAN pseudotime) before running
  lamian_test() for trajectory differential expression or density testing.
when_to_use_negative:
- Input is already a preprocessed Lamian test object or Res$statistics table — filtering
  has already been applied upstream
- Single-sample trajectories where sample-level filtering (minimum cells per sample)
  is meaningless and cross-sample variability does not need accounting
- You need Modules 1-2 (trajectory construction/topology testing) compatibility with
  Monocle2/Monocle3/Slingshot-internal structures — the article notes these modules
  do not seamlessly interoperate with those trajectory objects
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2229
tools:
- name: Lamian
  role: Framework consuming the preprocessed expr/cellanno/pseudotime/design objects
    in downstream TDE/XDE/TCD/XCD tests; installed from GitHub
  repo: https://github.com/Winnie09/Lamian
- name: Slingshot
  role: Source of the per-cell pseudotime values that must be subsetted (NA cells
    dropped) during preprocessing
- name: TSCAN
  role: Default trajectory/pseudotime construction method in Lamian; its GitHub version
    must be installed fresh (remove old TSCAN) during setup
- name: scRNA-seq
  role: 'Input data modality: raw count matrices whose cells and genes are quality-filtered
    in this preprocessing step'
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-023-42841-y
    title: A statistical framework for differential pseudotime analysis with multiple
      single-cell RNA-seq samples
  source_content_hash: sha256:07e67ce2810251bc30dd090764f88db80476675ec570061ec70c70f5615c73c2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Winnie09/Lamian
  tools:
  - Lamian
  - Slingshot
  - TSCAN
  - scRNA-seq
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# pseudotime-lineage-preprocessing-and-filtering

## Summary

Preprocess scRNA-seq data and pseudotime before differential pseudotime testing with Lamian: quality-filter cells and genes, drop cells lacking pseudotime, and exclude under-powered samples so that downstream TDE/XDE/TCD/XCD tests (e.g., on the HCA bone marrow myeloid/erythroid/lymphoid lineages) have adequate signal-to-noise.

## When to use

Use when preparing multi-sample scRNA-seq data (e.g., HCA bone marrow counts with cell-to-sample annotation and Slingshot/TSCAN pseudotime) before running lamian_test() for trajectory differential expression or density testing.

## When NOT to use

- Input is already a preprocessed Lamian test object or Res$statistics table — filtering has already been applied upstream
- Single-sample trajectories where sample-level filtering (minimum cells per sample) is meaningless and cross-sample variability does not need accounting
- You need Modules 1-2 (trajectory construction/topology testing) compatibility with Monocle2/Monocle3/Slingshot-internal structures — the article notes these modules do not seamlessly interoperate with those trajectory objects

## Inputs

- Raw gene expression count matrix from HCA-BM 10x Genomics scRNA-seq data (e.g., hca_bm_saver example object)
- Cell-to-sample annotation table (cellanno, e.g., hca_bm_cellanno)
- Per-cell pseudotime values from Slingshot/TSCAN trajectory inference (e.g., slingshot/expdata objects), possibly containing NA for cells off-lineage
- Design matrix with sample covariates (intercept plus variables such as sex)
- Per-cell QC metrics: read counts, number of expressed genes, fraction of mitochondrial reads

## Outputs

- Filtered expression matrix, cellanno, pseudotime, and design objects ready for lamian_test()
- Per-lineage preprocessed subsets (myeloid, erythroid, lymphoid) with cells having non-NA pseudotime and samples with >= 50 cells

## How to apply

Start from the expression count matrix, cell-to-sample annotation (cellanno), pseudotime vector, and design matrix, then first apply cell-level quality filters — in the HCA-BM data, cells with fewer than 5000 reads, fewer than 1000 expressed genes, or more than 10% mitochondrial reads are removed, and genes expressed in less than 0.1% of all cells are dropped. Next, subset pseudotime: drop cells with NA pseudotime values along the lineage of interest. Then exclude samples with too few cells to support reliable testing — fewer than 50 cells per sample for the HCA-BM workflow (the COVID analyses use 500 cells and 100 CD8+ T cells as analogous cutoffs). Keep the first design column as the intercept; only after this filtering should lamian_test() be run per lineage with test.type='time' or test.type='variable' (testvar=2, permuiter=100).

## Related tools

- **Lamian** (Framework consuming the preprocessed expr/cellanno/pseudotime/design objects in downstream TDE/XDE/TCD/XCD tests; installed from GitHub) — https://github.com/Winnie09/Lamian
- **Slingshot** (Source of the per-cell pseudotime values that must be subsetted (NA cells dropped) during preprocessing)
- **TSCAN** (Default trajectory/pseudotime construction method in Lamian; its GitHub version must be installed fresh (remove old TSCAN) during setup)
- **scRNA-seq** (Input data modality: raw count matrices whose cells and genes are quality-filtered in this preprocessing step)

## Examples

```
library(Lamian); data(slingshot_data, expdata); keep <- !is.na(pt) & (table(cellanno[,2])[cellanno[,2]] >= 50); res <- lamian_test(expr[keep,], cellanno[keep,], pseudotime[keep], design=design, test.type='time', permuiter=100)
```

## Evaluation signals

- No NA pseudotime values remain in the filtered per-lineage pseudotime vector
- Every retained sample has at least 50 cells in the HCA-BM workflow (analogous cutoffs: 500 cells / 100 CD8+ T cells in COVID analyses)
- No cell with fewer than 5000 reads, fewer than 1000 expressed genes, or > 10% mitochondrial reads survives; no gene expressed in < 0.1% of cells remains
- Downstream lamian_test() runs reproduce the published significance counts (TDE 8475/7454/8953 and XDE 43/32/29 for myeloid/erythroid/lymphoid at fdr.overall < 0.05)
- Known lineage markers (CD14, HBB, GATA1, CD3D, CD19, CD27) are recoverable in the filtered gene set

## Limitations

- The minimum-cells-per-sample threshold is dataset-specific (50 for HCA-BM; 500 cells / 100 CD8+ T cells for COVID) and must be re-justified for new datasets
- Lamian modules 1 and 2 do not seamlessly connect to Monocle2/Monocle3/Slingshot trajectory objects, so preprocessed inputs must match the expected Lamian data structures (e.g., hca_bm_saver, hca_bm_pca, hca_bm_cellanno)
- Preprocessing cannot compensate for pseudotime inference uncertainty; jointly accounting for pseudotime uncertainty with gene expression and cell abundance uncertainty is deferred to future work

## Evidence

- [discussion] Cells with fewer than 5000 reads, fewer than 1,000 expressed genes (i.e. genes with nonzero read count), or more than 10% of reads mapped to the mitochondrial genome were deemed as low quality and: "Cells with fewer than 5000 reads, fewer than 1,000 expressed genes (i.e. genes with nonzero read count), or more than 10% of reads mapped to the mitochondrial genome were deemed as low quality and"
- [discussion] We also filtered out genes that were expressed in less than 0.1% of all cells.: "We also filtered out genes that were expressed in less than 0.1% of all cells."
- [methods] we will use an example dataset **expdata** of 100 genes and 1000 cells to demonstration the workflow: "we will use an example dataset **expdata** of 100 genes and 1000 cells to demonstration the workflow"
- [methods] Here, we use example data **hca_bm_saver**, **hca_bm_pca**, **hca_bm_cellanno** to demonstrate their data structures.: "Here, we use example data **hca_bm_saver**, **hca_bm_pca**, **hca_bm_cellanno** to demonstrate their data structures."
- [abstract] Lamian accounts for cross-sample variability, reducing sample-specific false discoveries not generalizable to new samples: "Unlike existing methods that ignore sample variability, Lamian draws statistical inference after accounting for cross-sample variability and hence substantially reduces sample-specific false"
- [readme] Users should have R installed on their computer before installing Lamian. R version needs to be at least 3.5.x or higher.: [evidence span withheld — anchor status: external_doc]
