---
name: signature-set-construction
description: Use when you need to define a new cell-type or process gene signature
  (curated marker gene list) or extend an existing signature set so it can be scored
  per cell with UCell, including over pre-calculated rankings.
when_to_use_negative:
- The input is already a per-cell signature score table rather than an expression
  matrix — scoring signatures again is unnecessary
- The analysis requires absolute expression quantification rather than rank-based
  enrichment — UCell scores depend only on relative gene ranks within each cell
- Genes in the proposed signature are absent from the expression matrix (e.g. low-depth
  data where markers like TPSAB1 are undetected) — scores will be uninformative
edam_operation: http://edamontology.org/operation_3232
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: UCell
  role: R package providing ScoreSignatures_UCell to compute Mann-Whitney U-based
    signature enrichment scores and StoreRankings_UCell to pre-compute reusable gene
    rankings
  repo: https://github.com/carmonalab/UCell
- name: R
  role: Environment in which signatures are defined as named gene lists and scoring
    is run
- name: SingleCellExperiment
  role: Bioconductor container for the gene-vs-cell matrix accepted by UCell scoring
    functions
- name: data.table
  role: Provides the frank function used internally by UCell for efficient rank computations
- name: future
  role: Enables parallel processing of rank/score batches to minimize execution time
- name: SignatuR
  role: Optional R package for retrieving and storing curated gene signatures to use
    as UCell input
  repo: https://github.com/carmonalab/SignatuR
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.csbj.2021.06.043
    title: 'UCell: Robust and scalable single-cell gene signature scoring'
  source_content_hash: sha256:2012dcbbab600619e0a26062bc3951189b01eb5282d716b18529be609155dd4b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/carmonalab/UCell
  tools:
  - R
  - SignatuR
  - SingleCellExperiment
  - UCell
  - data.table
  - future
  edam_operation: http://edamontology.org/operation_3232
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# signature-set-construction

## Summary

Construct and extend gene signature sets (named lists of marker genes) for scoring with UCell's Mann-Whitney U statistic, e.g. adding new signatures such as Mast.cell = c("TPSAB1","TPSB2","CPA3","MS4A2") and Lymphoid = c("LCK") to an existing signature list before scoring.

## When to use

Use when you need to define a new cell-type or process gene signature (curated marker gene list) or extend an existing signature set so it can be scored per cell with UCell, including over pre-calculated rankings.

## When NOT to use

- The input is already a per-cell signature score table rather than an expression matrix — scoring signatures again is unnecessary
- The analysis requires absolute expression quantification rather than rank-based enrichment — UCell scores depend only on relative gene ranks within each cell
- Genes in the proposed signature are absent from the expression matrix (e.g. low-depth data where markers like TPSAB1 are undetected) — scores will be uninformative

## Inputs

- Gene-vs-cell single-cell expression matrix (sparse matrix or SingleCellExperiment/Seurat object), e.g. the downsampled Zilionis dataset
- Curated marker gene lists as named character vectors of gene symbols (e.g. Mast.cell, Lymphoid, Tcell_signature)
- Optional pre-calculated rank matrix from StoreRankings_UCell (precalc.ranks)

## Outputs

- Named list of gene signatures combining original and newly added sets
- Per-cell signature score table (one score per cell per signature), e.g. written to ucell_scores.csv

## How to apply

Assemble each signature as a named list of gene symbols in R (e.g. Mast.cell = c("TPSAB1","TPSB2","CPA3","MS4A2"), Lymphoid = c("LCK")) and append it to the original signature set so all signatures — old and new — are scored in a single pass. Score the signatures with ScoreSignatures_UCell on a gene-vs-cell expression matrix or, for efficiency, on rankings pre-computed once with StoreRankings_UCell using the default rmax = 1500 cap (genes ranked above rmax receive rank rmax + 1). Because UCell scores depend only on the relative gene expression ranks within individual cells, a signature defined this way yields scores robust to dataset composition, unlike Seurat's AddModuleScore which bins genes by dataset-wide average expression. UCell also supports positive and negative gene sets within a signature by appending + or - signs to gene symbols (e.g. c("CD2+","CD8A+","CD4-")). Verify the resulting per-cell score distributions are consistent with expected ranges (scores bounded by 0 and 1 via UCell = 1 - U/Umax) before downstream use.

## Related tools

- **UCell** (R package providing ScoreSignatures_UCell to compute Mann-Whitney U-based signature enrichment scores and StoreRankings_UCell to pre-compute reusable gene rankings) — https://github.com/carmonalab/UCell
- **R** (Environment in which signatures are defined as named gene lists and scoring is run)
- **SingleCellExperiment** (Bioconductor container for the gene-vs-cell matrix accepted by UCell scoring functions)
- **data.table** (Provides the frank function used internally by UCell for efficient rank computations)
- **future** (Enables parallel processing of rank/score batches to minimize execution time)
- **SignatuR** (Optional R package for retrieving and storing curated gene signatures to use as UCell input) — https://github.com/carmonalab/SignatuR

## Examples

```
precalc.ranks <- StoreRankings_UCell(expr.matrix); gene.sets <- c(orig.sets, list(Mast.cell=c("TPSAB1","TPSB2","CPA3","MS4A2"), Lymphoid=c("LCK"))); scores <- ScoreSignatures_UCell(precalc.ranks, features=gene.sets)
```

## Evaluation signals

- Score distributions and ranges of newly added signatures (e.g. Mast.cell, Lymphoid) are consistent with values reported in the basic UCell vignette
- All scores fall in [0, 1] after the U' = 1 - U/Umax normalization with rmax = 1500
- Per-cell score table contains one column per signature and one row per cell, matching the cell names of the input matrix
- Re-scoring original signatures over stored rankings reproduces the original scores computed directly from the expression matrix
- UCell scores correlate with AUC-based methods such as AUCell, as expected from the relation between the U statistic and the AUC

## Limitations

- The rmax parameter (default 1500) caps informative ranks; genes ranked above rmax are assigned rmax + 1, losing resolution in the uninformative tail of zero-expression genes
- UCell normalization (U' = 1 - U/Umax, updated in version >= 2.7.6) changed from earlier Umax = n*maxRank; for typical applications results are similar but large gene sets show more homogeneous score distributions in newer versions
- Single-cell data sparsity can make scores noisy; SmoothKNN can partially correct this by weighted averaging over k nearest neighbors
- Scores are rank-based and therefore insensitive to absolute expression magnitude differences between datasets

## Evidence

- [abstract] UCell is an R package for evaluating gene signatures in single-cell datasets.: "UCell is an R package for evaluating gene signatures in single-cell datasets."
- [methods] cap ranks above rmax to rmax+1: "we set rg,c = rmax + 1 for all rg,c > rmax, with rmax = 1500 by default"
- [methods] scores robust across datasets: "This score depends only on the gene expression ranks of individual cell, and therefore is robust across datasets."
- [methods] StoreRankings_UCell function: "`StoreRankings_UCell`   Calculate and store gene rankings for a single-cell dataset."
- [readme] positive and negative gene sets supported: "Note that UCell supports **positive and negative gene sets** within a signature. Simply append + or - signs to the genes"
- [intro] Seurat AddModuleScore inconsistency motivates rank-based signatures: "because genes are binned based on their average expression across the whole dataset for normalization purposes, the method generates inconsistent results"
