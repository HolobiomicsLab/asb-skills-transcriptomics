---
name: single-cell-differential-expression-testing
description: Use when you have a gene-by-cell expression matrix with cell type/cluster
  assignments and need to identify cell type-specific DE genes or TFs (e.g., Welch's
  t test p < 0.05) to feed signature or regulatory-network analysis.
when_to_use_negative:
- Cell types or clusters have not yet been defined — SINCERA requires cell type identification
  before DE testing, since specificity is computed per cell type against all others.
- Genes are non- or low-expressed and have not passed prefiltering (prefilterGenes
  / expr.minimum), so t tests would be run on mostly-zero profiles.
- The goal is differential expression between two predefined biological conditions
  rather than cell type specificity within one scRNA-seq dataset — a two-condition
  DE tool is more appropriate.
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: SINCERA
  role: R S4 pipeline whose drivingfoce.selectTFs / drivingforce.selectTGs functions
    perform the per-cell-type one-tailed Welch's t test (p < 0.05) that produces the
    DE TF and TG candidate sets
  repo: https://github.com/xu-lab/SINCERA
- name: SAMstrt
  role: Dedicated single-cell differential expression analysis tool cited in the article's
    introduction as an alternative DE-testing approach
- name: SCDE
  role: Dedicated single-cell differential expression analysis tool cited in the article's
    introduction as an alternative DE-testing approach
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1007/978-1-4939-7710-9_15
    title: Single-Cell Transcriptome Analysis Using SINCERA Pipeline
  source_content_hash: sha256:b335e20bb0902b9edc6235b351a44f306b8c7e48a7dea6fc4bd32f8e8c7143f1
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/xu-lab/SINCERA
  tools:
  - SAMstrt
  - SCDE
  - SINCERA
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# single-cell-differential-expression-testing

## Summary

Test each gene's expression in one cell type against all other cells in a scRNA-seq dataset to flag cell type-specific differentially expressed genes and transcription factors (one-tailed Welch's t test, p < 0.05), as implemented in the SINCERA pipeline and echoed by dedicated scRNA-seq DE tools such as SAMstrt and SCDE. The resulting DE calls are the raw material for cell type-specific signature and driving-force analyses.

## When to use

Use when you have a gene-by-cell expression matrix with cell type/cluster assignments and need to identify cell type-specific DE genes or TFs (e.g., Welch's t test p < 0.05) to feed signature or regulatory-network analysis.

## When NOT to use

- Cell types or clusters have not yet been defined — SINCERA requires cell type identification before DE testing, since specificity is computed per cell type against all others.
- Genes are non- or low-expressed and have not passed prefiltering (prefilterGenes / expr.minimum), so t tests would be run on mostly-zero profiles.
- The goal is differential expression between two predefined biological conditions rather than cell type specificity within one scRNA-seq dataset — a two-condition DE tool is more appropriate.

## Inputs

- gene-by-cell expression matrix (exprfile) loaded into a SINCERA S4 object
- samplefile with cell/sample metadata
- cell type or cluster assignments for each cell (from SINCERA cell type identification)
- list of transcription factors to restrict TF-level DE testing

## Outputs

- per-cell-type table of differentially expressed genes with Welch's t test p-values (threshold p < 0.05)
- candidate TF set per cell type (union of DE TFs and TFs expressed in >= 80% of the cell type's cells)
- candidate target gene set per cell type (cell type-specific DE genes or signature genes)

## How to apply

Within SINCERA, first construct the R S4 object from the exprfile gene-by-cell expression matrix and samplefile, then preprocess (normalization.zscore per-sample z-score, prefilterGenes for non-/low-expressed genes) and define cell types before any DE testing. For each cell type, run a one-tailed Welch's t test of each gene's expression profile in that cell type versus all other cells, accepting genes with p-value < 0.05 as cell type-specific DE. For driving-force analysis specifically, run drivingfoce.selectTFs to take the union of cell type-specific differentially expressed TFs (p < 0.05) and commonly expressed TFs (expressed in at least 80% of the cell type's cells), and drivingforce.selectTGs to designate cell type-specific DE genes or signature genes as candidate target genes. The rationale is that DE testing supplies the candidate regulators and targets that the subsequent TRN inference (drivingforce.inferTRN, edge.threshold = 0.05) operates on; alternatively, dedicated single-cell DE tools like SAMstrt or SCDE can be used at this stage as noted in the article's introduction.

## Related tools

- **SINCERA** (R S4 pipeline whose drivingfoce.selectTFs / drivingforce.selectTGs functions perform the per-cell-type one-tailed Welch's t test (p < 0.05) that produces the DE TF and TG candidate sets) — https://github.com/xu-lab/SINCERA
- **SAMstrt** (Dedicated single-cell differential expression analysis tool cited in the article's introduction as an alternative DE-testing approach)
- **SCDE** (Dedicated single-cell differential expression analysis tool cited in the article's introduction as an alternative DE-testing approach)

## Examples

```
devtools::install_github("xu-lab/SINCERA"); library(SINCERA); de.genes <- drivingfoce.selectTFs(object = sc, tf.list = tfs)
```

## Evaluation signals

- Every accepted DE call has a one-tailed Welch's t test p-value < 0.05 for the focal cell type versus all other cells.
- The candidate TF set equals the union of cell type-specific DE TFs and TFs expressed in at least 80% of that cell type's cells — check both branches are present.
- Candidate target genes are exclusively cell type-specific DE genes or signature genes of the focal cell type.
- Downstream check: drivingforce.inferTRN's largest connected component must contain >= 80% of the selected TFs and TGs (drivingforce.getLCC), which fails if the DE candidate sets are degenerate.
- The final drivingforce.rankTFs table lists every TF in the LCC with its integrated importance score, confirming the DE-derived TF input was complete.

## Limitations

- The one-tailed Welch's t test only detects upregulation in the focal cell type; downregulated genes are not flagged.
- DE testing is downstream of preprocessing and cell type identification, so errors in clustering or contaminated/low-quality cells (filterLowQualityCells, filterContaminatedCells) propagate into the DE calls.
- SINCERA is designed to run on standard desktop/laptop computers, so very large datasets may be constrained.

## Evidence
