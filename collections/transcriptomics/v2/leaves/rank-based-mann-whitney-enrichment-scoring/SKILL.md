---
name: rank-based-mann-whitney-enrichment-scoring
description: Use when scoring cell-type or biological-process gene signatures on a
  gene-vs-cell scRNA-seq matrix, Seurat object, or SingleCellExperiment, and you need
  scores that are unaffected by dataset composition.
when_to_use_negative:
- The input is not single-cell expression data ranked per cell (e.g. already aggregated
  bulk or feature tables), where rank-capping at rmax=1500 is meaningless
- You need dataset-level normalization that depends on the full dataset composition
  — UCell scores deliberately depend only on relative expression within individual
  cells
- Signatures contain mostly genes absent from the reference, where 'impute' handling
  (setting absent genes to rmax+1) can distort scores
edam_operation: http://edamontology.org/operation_3933
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: UCell
  role: core R package computing Mann-Whitney U-based signature scores via ScoreSignatures_UCell,
    AddModuleScore_UCell, and StoreRankings_UCell
  repo: https://github.com/carmonalab/UCell
- name: data.table
  role: provides the frank function used internally for efficient per-cell gene ranking
- name: future
  role: enables parallel processing of automatically split data batches to reduce
    execution time
- name: Seurat
  role: object framework targeted by the AddModuleScore_UCell wrapper
  repo: https://github.com/satijalab/seurat/
- name: SingleCellExperiment
  role: Bioconductor container scored directly by ScoreSignatures_UCell, returning
    scores in altExp(sce, 'UCell')
- name: pyUCell
  role: Python implementation of the same UCell scoring method
  repo: https://github.com/carmonalab/pyucell
provenance:
  source_task_ids:
  - task_003
  - task_006
  - task_005
  - task_001
  - task_002
  source_papers:
  - doi: 10.1016/j.csbj.2021.06.043
    title: 'UCell: Robust and scalable single-cell gene signature scoring'
  source_content_hash: sha256:6a337063361f790c427986e3d147fe74bd7b5e813bbfe6088c0c056ebfdf7917
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
  - pyUCell
  edam_operation: http://edamontology.org/operation_3933
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# rank-based-mann-whitney-enrichment-scoring

## Summary

Score gene signatures in single-cell data using UCell, which computes a normalized Mann-Whitney U statistic (equivalent to AUC) on per-cell gene expression ranks, making scores robust to dataset size and composition.

## When to use

Use when scoring cell-type or biological-process gene signatures on a gene-vs-cell scRNA-seq matrix, Seurat object, or SingleCellExperiment, and you need scores that are unaffected by dataset composition.

## When NOT to use

- The input is not single-cell expression data ranked per cell (e.g. already aggregated bulk or feature tables), where rank-capping at rmax=1500 is meaningless
- You need dataset-level normalization that depends on the full dataset composition — UCell scores deliberately depend only on relative expression within individual cells
- Signatures contain mostly genes absent from the reference, where 'impute' handling (setting absent genes to rmax+1) can distort scores

## Inputs

- gene-vs-cell expression matrix (sparse or dense), e.g. the bundled sample.matrix or Zilionis gene-by-cell counts
- Seurat object (e.g. downsampled Zilionis NSCLC immune-cell subset)
- SingleCellExperiment object with a 'counts' assay
- named list of gene signatures, optionally with positive/negative gene sets (e.g. c("CD2+","CD8A+","CD4-"))

## Outputs

- per-cell signature score table (one column per signature, one row per cell) with normalized U statistics in [0,1]
- altExp(sce, 'UCell') object when scoring a SingleCellExperiment
- pre-calculated rank matrix from StoreRankings_UCell for reuse with new signatures

## How to apply

Rank genes per cell by sorting each column of the expression matrix (UCell uses data.table::frank), capping ranks above rmax = 1500 at rmax + 1 to mitigate the uninformative tail of zero-expression genes. For a signature of n genes, compute the Mann-Whitney U statistic from the signature genes' ranks and normalize as UCell = 1 - U/U_max, where U_max = n*maxRank - n(n+1)/2 (versions >= 2.7.6); scores are bounded [0,1] and correlate with AUC-based methods such as AUCell. Apply ScoreSignatures_UCell directly to matrices or SingleCellExperiment objects (scores returned in altExp(sce, 'UCell')), or AddModuleScore_UCell as the Seurat wrapper. For low-depth datasets (e.g. the downsampled Zilionis lung immune data), lower maxRank to ~800-1000 instead of the default 1500. For costly re-scoring with new signatures, pre-calculate rankings once with StoreRankings_UCell and run ScoreSignatures_UCell on the stored ranks for considerably faster score calculation. Handle missing signature genes with missing_genes='impute' (set to rmax+1) or 'skip' (drop from rank sum).

## Related tools

- **UCell** (core R package computing Mann-Whitney U-based signature scores via ScoreSignatures_UCell, AddModuleScore_UCell, and StoreRankings_UCell) — https://github.com/carmonalab/UCell
- **data.table** (provides the frank function used internally for efficient per-cell gene ranking)
- **future** (enables parallel processing of automatically split data batches to reduce execution time)
- **Seurat** (object framework targeted by the AddModuleScore_UCell wrapper) — https://github.com/satijalab/seurat/
- **SingleCellExperiment** (Bioconductor container scored directly by ScoreSignatures_UCell, returning scores in altExp(sce, 'UCell'))
- **pyUCell** (Python implementation of the same UCell scoring method) — https://github.com/carmonalab/pyucell

## Examples

```
scores <- ScoreSignatures_UCell(sample.matrix, features=list(Tcell_signature=c("CD2","CD3E","CD3D"), Myeloid_signature=c("SPI1","FCER1G","CSF1R"))); head(scores)
```

## Evaluation signals

- All returned scores are normalized U statistics (AUC-equivalent) bounded between 0 and 1
- Scoring the same cells on the full dataset versus a subset yields identical score distributions for those cells (robustness to dataset composition)
- Scores from pre-calculated rankings (StoreRankings_UCell + ScoreSignatures_UCell) match scores computed directly on the expression matrix
- For identical parameter settings, columns reproduce vignette reference values; differences appear only in columns affected by changed maxRank or missing_genes settings
- Output table has one row per cell (e.g. 5000 rows for the downsampled Zilionis subset) and one column per requested signature

## Limitations

- Default rmax=1500 rank cap may be too high for low-depth datasets; for the downsampled Zilionis data the vignette recommends maxRank around 800-1000
- Genes ranked above rmax are all assigned rank rmax + 1, so signatures whose genes fall entirely in the zero-expression tail get uninformative scores
- Single-cell sparsity may require SmoothKNN imputation of scores over k-nearest neighbors to partially correct dropouts
- Normalization changed in version >= 2.7.6 (U_max = n*maxRank - n(n+1)/2 vs the earlier n*maxRank), so scores are not strictly comparable across versions for large gene sets

## Evidence

- [intro] UCell, a gene signature scoring method based on the Mann-Whitney U statistic whose scores depend only on relative gene expression in individual cells: "we propose UCell, a gene signature scoring method based on the Mann-Whitney U statistic. UCell scores depend only on the relative gene expression in individual cells"
- [methods] rank capping at rmax=1500 with rmax+1 assignment: "we set rg,c = rmax + 1 for all rg,c > rmax, with rmax = 1500 by default"
- [methods] frank ranking implementation: "Internally, UCell uses the frank function from the data.table package [9] for efficient ranks computations."
- [readme] new normalization formula and rationale: [evidence span withheld — anchor status: external_doc]
- [readme] positive and negative gene set support: "Note that UCell supports **positive and negative gene sets** within a signature. Simply append + or - signs to the genes"
- [results] robustness across dataset subsets: "Evaluating a CD8 T cell signature on the full dataset or on CD8 T cells only, results in identical score distributions for CD8 T cells in the two settings"
