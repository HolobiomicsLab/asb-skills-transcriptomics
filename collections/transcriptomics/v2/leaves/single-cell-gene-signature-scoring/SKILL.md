---
name: single-cell-gene-signature-scoring
description: Use when you need per-cell enrichment scores for cell-type or process
  gene signatures from scRNA-seq data (matrix, Seurat, or SingleCellExperiment input)
  and require scores robust to dataset heterogeneity.
when_to_use_negative:
- You need absolute expression quantification rather than rank-based per-cell enrichment
  — UCell scores depend only on relative gene expression ranks within individual cells
- The task is cell-type annotation/purification from marker genes rather than signature
  scoring — use scGate, which builds upon UCell for that purpose
- You only need signature databases rather than scoring — use SignatuR to store/retrieve
  gene signatures instead
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: UCell
  role: primary R package computing Mann-Whitney U-based signature scores; provides
    ScoreSignatures_UCell, AddModuleScore_UCell, StoreRankings_UCell, SmoothKNN
  repo: https://github.com/carmonalab/UCell
- name: R
  role: implementation language for the UCell package and scoring workflows
- name: Seurat
  role: object framework that AddModuleScore_UCell interacts with directly
  repo: https://github.com/satijalab/seurat/
- name: SingleCellExperiment
  role: Bioconductor container scored directly by ScoreSignatures_UCell; scores returned
    in altExp 'UCell'
- name: data.table
  role: its frank function performs efficient per-cell gene ranking inside UCell
- name: future
  role: enables parallel processing of rank batches for large datasets
provenance:
  source_task_ids:
  - task_001
  - task_003
  - task_004
  - task_005
  - task_006
  - task_002
  source_papers:
  - doi: 10.1016/j.csbj.2021.06.043
    title: 'UCell: Robust and scalable single-cell gene signature scoring'
  source_content_hash: sha256:db79aff7a6b95c5e47fc356cda07503a77b7540c7a24dfda987fe74f3ab2c4ef
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
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# single-cell-gene-signature-scoring

## Summary

Score gene signatures (e.g., Tcell, Myeloid, NK, Plasma_cell) in single-cell RNA-seq datasets with UCell, whose Mann-Whitney U statistic scores depend only on relative gene expression per cell and are robust to dataset size and composition. Scores are computable from gene-by-cell matrices, Seurat objects, or SingleCellExperiment objects.

## When to use

Use when you need per-cell enrichment scores for cell-type or process gene signatures from scRNA-seq data (matrix, Seurat, or SingleCellExperiment input) and require scores robust to dataset heterogeneity.

## When NOT to use

- You need absolute expression quantification rather than rank-based per-cell enrichment — UCell scores depend only on relative gene expression ranks within individual cells
- The task is cell-type annotation/purification from marker genes rather than signature scoring — use scGate, which builds upon UCell for that purpose
- You only need signature databases rather than scoring — use SignatuR to store/retrieve gene signatures instead

## Inputs

- gene-by-cell expression matrix (sparse or dense, e.g., UCell's bundled sample.matrix)
- named list of gene signatures (with optional +/- positive/negative gene markers)
- Seurat object (e.g., downsampled Zilionis NSCLC immune-cell subset)
- SingleCellExperiment object with counts in the 'counts' assay

## Outputs

- per-cell signature score table (one row per cell, one column per signature; normalized U statistic in [0,1])
- Seurat object metadata columns prefixed per signature after AddModuleScore_UCell
- altExp(sce, 'UCell') object holding per-cell scores for SingleCellExperiment input

## How to apply

Assemble a named list of signatures (supporting positive/negative genes like 'CD3D-' via appended +/- signs) and run UCell on your input: ScoreSignatures_UCell for gene-vs-cell matrices or SingleCellExperiment objects, or AddModuleScore_UCell as the Seurat wrapper. Internally, UCell uses data.table::frank to rank genes per cell, caps all ranks above rmax = 1500 at rmax + 1 to mitigate the uninformative zero-expression tail, and computes the normalized Mann-Whitney U statistic (AUC-equivalent, bounded [0,1]); on SingleCellExperiment input scores are returned in altExp(sce, 'UCell'). For low-depth datasets, lower the rank cap (e.g., maxRank 800-1000 instead of 1500); handle signature genes absent from the data with missing_genes='impute' (set to rmax+1) or 'skip' (drop from rank sum). Large datasets are automatically split into batches processed serially or in parallel via the future package; for repeated scoring, pre-calculate rankings once with StoreRankings_UCell and re-score new signatures over the stored ranks for considerably faster computation.

## Related tools

- **UCell** (primary R package computing Mann-Whitney U-based signature scores; provides ScoreSignatures_UCell, AddModuleScore_UCell, StoreRankings_UCell, SmoothKNN) — https://github.com/carmonalab/UCell
- **R** (implementation language for the UCell package and scoring workflows)
- **Seurat** (object framework that AddModuleScore_UCell interacts with directly) — https://github.com/satijalab/seurat/
- **SingleCellExperiment** (Bioconductor container scored directly by ScoreSignatures_UCell; scores returned in altExp 'UCell')
- **data.table** (its frank function performs efficient per-cell gene ranking inside UCell)
- **future** (enables parallel processing of rank batches for large datasets)

## Examples

```
library(UCell); gene.sets <- list(Tcell_signature=c("CD2","CD3E","CD3D"), Myeloid_signature=c("SPI1","FCER1G","CSF1R")); scores <- ScoreSignatures_UCell(sample.matrix, features=gene.sets)
```

## Evaluation signals

- All scores are normalized U statistics (AUC-equivalent) bounded between 0 and 1
- Output table contains one row per cell and one column per requested signature (e.g., 5000 rows for the downsampled Zilionis subset with Tcell, Myeloid, NK, Plasma_cell columns)
- Scores for identical parameter settings reproduce the vignette reference values; only columns affected by changed maxRank or missing_genes settings differ between runs
- For SingleCellExperiment input, the returned altExp named 'UCell' contains scores for all requested signatures
- A CD8 T cell signature yields identical score distributions whether evaluated on the full dataset or on CD8 T cells only

## Limitations

- U scores are capped at rmax = 1500 by default, which mitigates the zero-expression tail but requires adjustment (e.g., maxRank ~800-1000) for low-depth datasets
- Sparse single-cell data can produce scores that benefit from kNN imputation via SmoothKNN
- Earlier UCell implementations (before v2.7.6) used U_max = n * maxRank normalization; results may differ slightly for large gene sets versus the updated normalization

## Evidence

- [methods] UCell scores depend only on the gene expression ranks of individual cells and are therefore robust across datasets.: "This score depends only on the gene expression ranks of individual cell, and therefore is robust across datasets."
- [methods] Rank cap at rmax plus one for zero-expression tail.: "we set rg,c = rmax + 1 for all rg,c > rmax, with rmax = 1500 by default"
- [methods] data.table frank used for efficient ranking.: "Internally, UCell uses the frank function from the data.table package [9] for efficient ranks computations."
- [readme] UCell install and quick-start test from README.: "scores <- ScoreSignatures_UCell(sample.matrix, features=gene.sets)"
- [methods] U statistic closely related to AUC.: "the U statistic is closely related to the area-under-the-curve (AUC) statistic for ROC curves [8], therefore we expect UCell scores to correlate with methods based on AUC scores such as AUCell"
- [methods] Large datasets batched serially or in parallel.: "Large datasets are automatically split into batches of reduced size, which can be processed serially (minimizing memory usage) or in parallel"
