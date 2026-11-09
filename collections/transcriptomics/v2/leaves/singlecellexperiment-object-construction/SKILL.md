---
name: singlecellexperiment-object-construction
description: Use when raw gene-by-cell scRNA-seq count data (e.g. a matrix loaded
  with data.table) must be wrapped into a SingleCellExperiment with a 'counts' assay
  as input to UCell signature scoring and downstream UMAP plotting.
when_to_use_negative:
- Input is already a Seurat object — use AddModuleScore_UCell instead, which interacts
  directly with Seurat objects
- The data is already a SingleCellExperiment with a 'counts' assay — proceed directly
  to ScoreSignatures_UCell without rebuilding the object
- The input is a precomputed feature/score table rather than raw gene-by-cell counts
  — UCell ranks cannot be computed from non-count matrices
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: SingleCellExperiment
  role: Container class the count matrix is wrapped into (counts in the 'counts' assay)
    so UCell can score directly on the object and store results in altExp(sce, 'UCell')
- name: UCell
  role: ScoreSignatures_UCell() scores the gene signatures on the SCE using the Mann-Whitney
    U statistic and returns normalized U scores in the 'UCell' altExp
  repo: https://github.com/carmonalab/UCell
- name: data.table
  role: Used internally by UCell (frank function) for efficient rank computations;
    also useful for loading the count matrix
- name: future
  role: Enables parallel processing of rank/score computation for large datasets
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.csbj.2021.06.043
    title: 'UCell: Robust and scalable single-cell gene signature scoring'
  source_content_hash: sha256:63bc2752fafed97a7a6c184bc4dcb596559f4e9bf4fc577c7c304e5672dfb0b5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/carmonalab/UCell
  tools:
  - R
  - SingleCellExperiment
  - UCell
  - data.table
  - future
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# singlecellexperiment-object-construction

## Summary

Construct a Bioconductor SingleCellExperiment (SCE) object from a gene-by-cell count matrix with counts stored in the 'counts' assay, so UCell::ScoreSignatures_UCell can be run directly on the object and per-cell Mann-Whitney U signature scores are returned in altExp(sce, 'UCell').

## When to use

Use when raw gene-by-cell scRNA-seq count data (e.g. a matrix loaded with data.table) must be wrapped into a SingleCellExperiment with a 'counts' assay as input to UCell signature scoring and downstream UMAP plotting.

## When NOT to use

- Input is already a Seurat object — use AddModuleScore_UCell instead, which interacts directly with Seurat objects
- The data is already a SingleCellExperiment with a 'counts' assay — proceed directly to ScoreSignatures_UCell without rebuilding the object
- The input is a precomputed feature/score table rather than raw gene-by-cell counts — UCell ranks cannot be computed from non-count matrices

## Inputs

- gene-by-cell scRNA-seq count matrix (e.g. loaded from file via data.table)
- named list of gene signatures (signature name -> vector of signature genes)

## Outputs

- SingleCellExperiment object with counts in the 'counts' assay
- altExp(sce, 'UCell') containing per-cell signature scores in [0,1] for each signature
- UMAP embedding with per-cell signature score overlays (e.g. saved as ucell_signature_umap.png)
- signature score table for all cells and signatures

## How to apply

Load the gene-by-cell count matrix (e.g. with data.table) and build a SingleCellExperiment object with the data stored in the 'counts' assay. Define gene signatures (e.g. Tcell, Myeloid, NK, Plasma_cell) as named lists of signature genes, which UCell accepts alongside the SCE object. Call UCell::ScoreSignatures_UCell directly on the SCE with assay='counts' and the default rank cap rmax=1500; internally UCell computes relative gene-expression ranks per cell (using data.table::frank, optionally parallelized with the future package) and converts ranks to Mann-Whitney U-based scores. The normalized U scores (AUC-equivalent, bounded [0,1]) for each signature are returned as an altExp object named 'UCell' on the SCE. Confirm the altExp contains one score column per signature for all cells before computing a UMAP embedding and plotting per-cell signature scores on it.

## Related tools

- **SingleCellExperiment** (Container class the count matrix is wrapped into (counts in the 'counts' assay) so UCell can score directly on the object and store results in altExp(sce, 'UCell'))
- **UCell** (ScoreSignatures_UCell() scores the gene signatures on the SCE using the Mann-Whitney U statistic and returns normalized U scores in the 'UCell' altExp) — https://github.com/carmonalab/UCell
- **data.table** (Used internally by UCell (frank function) for efficient rank computations; also useful for loading the count matrix)
- **future** (Enables parallel processing of rank/score computation for large datasets)

## Examples

```
sce <- SingleCellExperiment(assays=list(counts=mat)); sce <- UCell::ScoreSignatures_UCell(sce, features=gene.sets, assay='counts'); ucell.scores <- altExp(sce, 'UCell')
```

## Evaluation signals

- altExp(sce, 'UCell') exists and contains score columns for all defined signatures (e.g. Tcell, Myeloid, NK, Plasma_cell)
- All UCell scores are normalized U statistics bounded in [0,1]
- Number of cells in the altExp scores matches the number of columns (cells) in the 'counts' assay of the SCE
- Score computation used assay='counts' on the SCE with default rank cap rmax=1500
- UMAP figure file (e.g. ucell_signature_umap.png) is generated and cells are colored by per-cell signature scores

## Limitations

- Scores depend on the rank cap parameter (rmax=1500 by default); genes ranked above rmax are set to rmax+1 to mitigate the uninformative tail of zero-expression genes
- Normalization of UCell scores changed in version >= 2.7.6 (Umax = n*maxRank - n(n+1)/2), so scores may differ slightly from earlier implementations for large gene sets
- Single-cell data are sparse; scores may benefit from kNN smoothing (SmoothKNN) on a dimensionality reduction such as PCA

## Evidence

- [readme] UCell can be applied to any cell vs. gene data matrix, and includes functions to directly interact with Seurat and Bioconductor's SingleCellExperiment objects.: "UCell can be applied to any cell vs. gene data matrix, and includes functions to directly interact with Seurat and Bioconductor's SingleCellExperiment objects."
- [methods] Given a gene vs. cell matrix (either as sparse matrix or stored in a `SingleCellExperiment` object): "Given a gene vs. cell matrix (either as sparse matrix or stored in a `SingleCellExperiment` object)"
- [methods] we first calculate the matrix of relative ranks R by sorting each column in X; in other words, we calculate a ranked list of genes for each cell in the dataset.: "we first calculate the matrix of relative ranks R by sorting each column in X; in other words, we calculate a ranked list of genes for each cell in the dataset."
- [results] UCell scores are based on the relative ranking of genes for individual cells, therefore they are robust to dataset composition.: "UCell scores are based on the relative ranking of genes for individual cells, therefore they are robust to dataset composition."
