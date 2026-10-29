---
name: single-cell-rna-seq-embryo-trajectory-analysis
description: Use when you have a merged, quantile-normalized single-cell expression
  matrix from a temporal biological process (e.g. mouse pre-implantation stages, oocyte
  to late blastocyst) and need pseudotime ordering that reflects developmental progression.
when_to_use_negative:
- Goal is ligand-receptor-to-TF causal inference between defined cell types (e.g.
  EPI vs PE) — use the full TimeTalk RunTimeTalk workflow instead of trajectory ordering
  alone
- Cells come from a process without temporal/developmental structure (definitive cell
  types, no stage ordering) — pseudotime from DDRTree would not reflect true progression
- Workflow requires monocle3 (e.g. RunMonocle3 on a Seurat object with minimal_branch_len)
  rather than the monocle2 DDRTree pipeline used here
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: monocle2
  role: 'Core trajectory engine: differentialGeneTest (qval = 0.01), reduceDimension
    with DDRTree, and orderCells produce the pseudotime ordering'
- name: R
  role: Runtime environment for building the CellDataSet and running the monocle2
    pipeline (analyses performed in R version 3.6.3)
- name: preprocessCore
  role: Provides normalize.quantiles (version 1.48.0) to produce the quantile-normalized
    expression matrix that feeds the CellDataSet
- name: TimeTalk
  role: Reference workflow package whose data-processing and trajectory step embeds
    this monocle2 ordering
  repo: https://github.com/ChengLiLab/TimeTalk
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s42003-023-05283-2
    title: TimeTalk uses single-cell RNA-seq datasets to decipher cell-cell communication
      during early embryo development
  source_content_hash: sha256:317b802cccb498039d026415b026286602a1d423a6f9639dbda8ea70f25de4f3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ChengLiLab/TimeTalk
  tools:
  - R
  - TimeTalk
  - monocle2
  - preprocessCore
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# single-cell-rna-seq-embryo-trajectory-analysis

## Summary

Reconstructs a developmental pseudotime trajectory from curated pre-implantation mouse embryo scRNA-seq data using monocle2: differential gene selection (qval = 0.01), DDRTree dimension reduction, and orderCells ordering of cells from oocyte to late blastocyst. This validates that merged, quantile-normalized single-cell data recapitulate early embryo development.

## When to use

Use when you have a merged, quantile-normalized single-cell expression matrix from a temporal biological process (e.g. mouse pre-implantation stages, oocyte to late blastocyst) and need pseudotime ordering that reflects developmental progression.

## When NOT to use

- Goal is ligand-receptor-to-TF causal inference between defined cell types (e.g. EPI vs PE) — use the full TimeTalk RunTimeTalk workflow instead of trajectory ordering alone
- Cells come from a process without temporal/developmental structure (definitive cell types, no stage ordering) — pseudotime from DDRTree would not reflect true progression
- Workflow requires monocle3 (e.g. RunMonocle3 on a Seurat object with minimal_branch_len) rather than the monocle2 DDRTree pipeline used here

## Inputs

- merged, quantile-normalized single-cell expression matrix (preprocessCore normalize.quantiles, version 1.48.0)
- per-cell developmental-stage labels for the 262 pre-implantation cells (oocyte through late blastocyst)

## Outputs

- monocle2 CellDataSet object with DDRTree reduced dimensions
- pseudotime-ordered trajectory containing all cells ordered from oocyte to late blastocyst
- trajectory plot reproducing the Fig. 1e developmental-stage arrangement

## How to apply

Start from the merged expression matrix produced with preprocessCore 'normalize.quantiles' (version 1.48.0) and build a monocle2 CellDataSet object containing all cells (262 pre-implantation cells in the reference workflow). Run monocle2 'differentialGeneTest' on the CellDataSet and retain only differentially expressed genes at cutoff qval = 0.01, since the developmentally variable genes should drive the trajectory geometry. Apply DDRTree dimension reduction on the selected genes via the monocle2 'reduceDimension' function to reconstruct the development trajectory. Call monocle2 'orderCells' to arrange all cells along pseudotime from oocyte to late blastocyst, then visualize/serialize the ordered trajectory for comparison against the expected developmental-stage arrangement (Fig. 1e layout in the reference study).

## Related tools

- **monocle2** (Core trajectory engine: differentialGeneTest (qval = 0.01), reduceDimension with DDRTree, and orderCells produce the pseudotime ordering)
- **R** (Runtime environment for building the CellDataSet and running the monocle2 pipeline (analyses performed in R version 3.6.3))
- **preprocessCore** (Provides normalize.quantiles (version 1.48.0) to produce the quantile-normalized expression matrix that feeds the CellDataSet)
- **TimeTalk** (Reference workflow package whose data-processing and trajectory step embeds this monocle2 ordering) — https://github.com/ChengLiLab/TimeTalk

## Examples

```
cds <- newCellDataSet(expr_matrix, expressionFamily=gaussianff()); cds <- differentialGeneTest(cds, qval=0.01); cds <- reduceDimension(cds, reduction_method="DDRTree"); cds <- orderCells(cds)
```

## Evaluation signals

- Output trajectory contains all 262 pre-implantation cells — no cell drop-out during differentialGeneTest or reduceDimension
- Cells are ordered along pseudotime from oocyte to late blastocyst, matching the developmental-stage arrangement of Fig. 1e
- Gene set used for DDRTree reduction is restricted to differentialGeneTest results at cutoff qval = 0.01
- Input matrix was quantile-normalized with preprocessCore normalize.quantiles (version 1.48.0) before CellDataSet construction

## Limitations

- Granger causality conclusions in the surrounding TimeTalk workflow are affected by parameter choices (e.g. winsz); users are advised to test various parameter combinations for dependable results
- Ligand-receptor-to-TF regulatory relationships inferred downstream of the trajectory are computational only and require wet-lab confirmation; scarcity of high temporal resolution datasets limits validation
- The trajectory reflects the curated public time-course mouse pre-implantation datasets (GEO refs 12,13); transferability depends on comparable temporal structure in new data

## Evidence

- [discussion] the differential expressed genes were used to perform DDRTree dimension reduction to reconstruct the development trajectory by the “reduceDimension” function.: "the differential expressed genes were used to perform DDRTree dimension reduction to reconstruct the development trajectory by the “reduceDimension” function."
- [discussion] we used the “differentialGeneTest” function to get differentially expressed genes (cutoff: qval = 0.01): "we used the “differentialGeneTest” function to get differentially expressed genes (cutoff: qval = 0.01)"
- [discussion] Quantile normalization was performed with the function “normalize.quantiles” from R package preprocessCore (version 1.48.0).: "Quantile normalization was performed with the function “normalize.quantiles” from R package preprocessCore (version 1.48.0)."
- [readme] devtools::install_github('cole-trapnell-lab/monocle3'): "devtools::install_github('cole-trapnell-lab/monocle3')"
