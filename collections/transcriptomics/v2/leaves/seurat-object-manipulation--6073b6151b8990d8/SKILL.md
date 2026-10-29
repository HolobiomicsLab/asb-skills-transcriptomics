---
name: seurat-object-manipulation
description: Use when a scRNA-seq dataset is stored as a Seurat object and you need
  to subset cells, downsample, or attach per-cell signature scores (e.g. Tcell, Myeloid,
  NK, Plasma_cell) via AddModuleScore_UCell.
when_to_use_negative:
- Input is a plain gene-vs-cell matrix or SingleCellExperiment rather than a Seurat
  object — use ScoreSignatures_UCell or the SingleCellExperiment interface instead
- You require scores that are independent of dataset composition via Seurat's own
  AddModuleScore, which bins genes on dataset-wide average expression and yields inconsistent
  per-cell results
- The analysis goal is not single-cell signature scoring (UCell is specific to scoring
  gene sets in scRNA-seq cells)
tools:
- name: UCell
  role: Provides AddModuleScore_UCell wrapper that computes Mann-Whitney U-based signature
    scores directly on Seurat objects
  repo: https://github.com/carmonalab/UCell
- name: Seurat
  role: Object framework holding the single-cell data being subset, downsampled, and
    annotated with UCell score columns
  repo: https://github.com/satijalab/seurat
- name: data.table
  role: Its frank function performs the per-cell gene ranking underlying the scores
    computed on the Seurat object
  repo: https://github.com/carmonalab/UCell
- name: future
  role: Enables serial or parallel processing of the automatically split batches when
    scoring large Seurat objects
- name: SingleCellExperiment
  role: Alternative Bioconductor object format supported by UCell when data is not
    in a Seurat object
provenance:
  source_task_ids:
  - task_001
  - task_005
  source_papers:
  - doi: 10.1016/j.csbj.2021.06.043
    title: 'UCell: Robust and scalable single-cell gene signature scoring'
  source_content_hash: sha256:2ee8a22ed22514565305e7d5e834e157e4b8aa9dc237ffe2cd08121070d0d79b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/carmonalab/UCell
  tools:
  - R
  - Seurat
  - SingleCellExperiment
  - UCell
  - data.table
  - future
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 15
---

# seurat-object-manipulation

## Summary

Manipulate Seurat objects to prepare single-cell datasets (loading, subsetting, downsampling, defining signature lists) and append per-cell gene-signature scores directly to the object using UCell's AddModuleScore_UCell Seurat wrapper.

## When to use

Use when a scRNA-seq dataset is stored as a Seurat object and you need to subset cells, downsample, or attach per-cell signature scores (e.g. Tcell, Myeloid, NK, Plasma_cell) via AddModuleScore_UCell.

## When NOT to use

- Input is a plain gene-vs-cell matrix or SingleCellExperiment rather than a Seurat object — use ScoreSignatures_UCell or the SingleCellExperiment interface instead
- You require scores that are independent of dataset composition via Seurat's own AddModuleScore, which bins genes on dataset-wide average expression and yields inconsistent per-cell results
- The analysis goal is not single-cell signature scoring (UCell is specific to scoring gene sets in scRNA-seq cells)

## Inputs

- Seurat object containing a gene-by-cell expression matrix (e.g. Zilionis NSCLC immune-cell subset downsampled to 5000 cells)
- Named list of gene signatures with optional +/- signed genes (e.g. Tcell, Myeloid, NK, Plasma_cell; CD8T/CD4 signatures)
- UCell scoring parameters (maxRank, chunk.size, w_neg, missing_genes)

## Outputs

- Seurat object with per-cell signature score columns added to cell metadata
- Per-cell score table (one column per signature, one row per cell, e.g. ucell_scores_zilionis.csv)
- Parameter-comparison table of score columns aligned by cell (e.g. ucell_parameter_comparison_scores.csv)

## How to apply

Load the scRNA-seq data (e.g. the Zilionis et al. 2019 NSCLC immune-cell dataset via the scRNAseq package), subset to the cells of interest, and downsample (e.g. to 5000 cells) inside the Seurat object. Assemble the gene signatures as a named list, optionally using +/- signs to mark positive/negative genes (e.g. c("CD2+","CD8A+","CD4-")). Run AddModuleScore_UCell directly on the Seurat object, tuning parameters for data depth: set maxRank≈800-1000 for low-depth data instead of the default rmax=1500 (the rank cap that mitigates the zero-expression tail), and set missing_genes='impute' (absent genes get rank rmax+1) or 'skip' (absent genes dropped from the rank sum). Large datasets are automatically split into batches, processed serially or in parallel through the future package; use chunk.size=500 for batched processing. Scores, computed via data.table::frank ranks and the normalized Mann-Whitney U statistic, are returned as metadata columns (one per signature) that can be extracted and written to CSV.

## Related tools

- **UCell** (Provides AddModuleScore_UCell wrapper that computes Mann-Whitney U-based signature scores directly on Seurat objects) — https://github.com/carmonalab/UCell
- **Seurat** (Object framework holding the single-cell data being subset, downsampled, and annotated with UCell score columns) — https://github.com/satijalab/seurat
- **data.table** (Its frank function performs the per-cell gene ranking underlying the scores computed on the Seurat object) — https://github.com/carmonalab/UCell
- **future** (Enables serial or parallel processing of the automatically split batches when scoring large Seurat objects)
- **SingleCellExperiment** (Alternative Bioconductor object format supported by UCell when data is not in a Seurat object)

## Examples

```
obj <- AddModuleScore_UCell(obj, features = gene.sets, maxRank = 1000, chunk.size = 500, w_neg = 1.0)
```

## Evaluation signals

- Output score table contains one row per downsampled cell (5000 rows for the Zilionis demo) and one column per requested signature (Tcell, Myeloid, NK, Plasma_cell)
- All scores are normalized U statistics (AUC-equivalent) bounded between 0 and 1
- For identical parameter settings, scores reproduce the vignette reference values; differences appear only in columns affected by changed maxRank or missing_genes settings
- Scores for a given cell are unchanged whether computed on the full dataset or on a cell subset (robustness to dataset composition)
- Absent genes (e.g. 'not_a_gene' in the Myeloid signature) are handled consistently with the chosen missing_genes mode ('impute' sets them to rmax+1; 'skip' drops them)

## Limitations

- Default maxRank=1500 is too permissive for low-depth datasets; the vignette recommends ~800-1000 (demonstrated with maxRank=1000) for the low-depth Zilionis data
- Genes absent from the dataset require an explicit missing_genes decision ('impute' vs 'skip'), which changes scores for affected signatures
- UCell version >= 2.7.6 changed the normalization to UCell = 1 - U/Umax with Umax = n*maxRank - n(n+1)/2; older implementations used Umax = n*maxRank, so scores may differ slightly across versions, especially for large gene sets

## Evidence

- [methods] AddModuleScore_UCell wrapper: "`AddModuleScore_UCell`   A wrapper for UCell to interact directly with Seurat objects."
- [methods] rank capping: "we set rg,c = rmax + 1 for all rg,c > rmax, with rmax = 1500 by default"
- [methods] batched parallel processing: "Large datasets are automatically split into batches of reduced size, which can be processed serially (minimizing memory usage) or in parallel"
- [readme] Seurat/SCE interop: "UCell can be applied to any cell vs. gene data matrix, and includes functions to directly interact with Seurat and Bioconductor's SingleCellExperiment objects."
- [readme] signed gene sets: "Note that UCell supports **positive and negative gene sets** within a signature. Simply append + or - signs to the genes to include them in positive and negative sets, respectively."
