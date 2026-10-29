---
name: ddrtree-dimensionality-reduction
description: Use when time-course scRNA-seq cells (e.g. sequential embryo stages such
  as oocyte, zygote, 2-cell through blastocyst) need to be arranged along a pseudotime
  development trajectory from a merged, quantile-normalized expression matrix.
when_to_use_negative:
- When the goal is cell-cell communication inference between defined cell types rather
  than pseudotime ordering — TimeTalk's eLR/Granger analysis (CellPhoneDB/CellChat-style)
  addresses that instead
- When cells are not from a time-course/dynamic process (no meaningful oocyte-to-blastocyst-like
  progression to order along pseudotime)
- When input is already a pseudotime-ordered or stage-resolved trajectory object requiring
  no dimension reduction
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: monocle2
  role: Provides differentialGeneTest (qval = 0.01 cutoff), reduceDimension with DDRTree,
    and orderCells for trajectory reconstruction and pseudotime ordering
- name: R
  role: Analysis environment in which the quantile-normalized matrix is loaded and
    the monocle2 CellDataSet workflow is executed (R version 3.6.3 used for statistical
    analyses)
  repo: https://github.com/ChengLiLab/TimeTalk
- name: preprocessCore
  role: Produces the quantile-normalized expression matrix (normalize.quantiles, version
    1.48.0) that serves as the trajectory workflow's input
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s42003-023-05283-2
    title: TimeTalk uses single-cell RNA-seq datasets to decipher cell-cell communication
      during early embryo development
  source_content_hash: sha256:f8813ac59618b6ef6bf2a1c955e43bfd18d0d40fe22f53d7132143ab5647906b
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
yes_rate: 0.8888888888888888
n_claims: 9
---

# ddrtree-dimensionality-reduction

## Summary

Reconstruct a developmental trajectory and pseudotime ordering for single-cell RNA-seq data by running monocle2's DDRTree dimension reduction on differentially expressed genes, then ordering cells with orderCells. In the TimeTalk paper this is applied to 262 mouse pre-implantation embryo cells to reproduce the oocyte-to-late-blastocyst trajectory (Fig. 1e).

## When to use

Use when time-course scRNA-seq cells (e.g. sequential embryo stages such as oocyte, zygote, 2-cell through blastocyst) need to be arranged along a pseudotime development trajectory from a merged, quantile-normalized expression matrix.

## When NOT to use

- When the goal is cell-cell communication inference between defined cell types rather than pseudotime ordering — TimeTalk's eLR/Granger analysis (CellPhoneDB/CellChat-style) addresses that instead
- When cells are not from a time-course/dynamic process (no meaningful oocyte-to-blastocyst-like progression to order along pseudotime)
- When input is already a pseudotime-ordered or stage-resolved trajectory object requiring no dimension reduction

## Inputs

- merged, quantile-normalized expression matrix (preprocessCore normalize.quantiles) for 262 mouse pre-implantation scRNA-seq cells
- developmental-stage labels (oocyte to late blastocyst) for each cell

## Outputs

- monocle2 CellDataSet object with DDRTree reduced dimensions
- pseudotime-ordered trajectory of all 262 cells from oocyte to late blastocyst
- trajectory plot reproducing the Fig. 1e developmental-stage layout

## How to apply

Load the merged, quantile-normalized expression matrix (normalized with preprocessCore 'normalize.quantiles', v1.48.0) into R and build a monocle2 CellDataSet object for all cells (262 pre-implantation cells in the paper). Run monocle2 'differentialGeneTest' and retain differentially expressed genes at the cutoff qval = 0.01, since only informative DE genes should drive the trajectory. Perform DDRTree dimension reduction on the selected genes using monocle2's 'reduceDimension' function to reconstruct the development trajectory, then call 'orderCells' to arrange all cells along pseudotime from oocyte to late blastocyst. Finally visualize/serialize the ordered trajectory so the pseudotime layout reproduces the developmental-stage arrangement of Fig. 1e.

## Related tools

- **monocle2** (Provides differentialGeneTest (qval = 0.01 cutoff), reduceDimension with DDRTree, and orderCells for trajectory reconstruction and pseudotime ordering)
- **R** (Analysis environment in which the quantile-normalized matrix is loaded and the monocle2 CellDataSet workflow is executed (R version 3.6.3 used for statistical analyses)) — https://github.com/ChengLiLab/TimeTalk
- **preprocessCore** (Produces the quantile-normalized expression matrix (normalize.quantiles, version 1.48.0) that serves as the trajectory workflow's input)

## Examples

```
cds <- reduceDimension(cds, reduction_method = "DDRTree"); cds <- orderCells(cds)
```

## Evaluation signals

- The output trajectory must contain all 262 cells ordered from oocyte to late blastocyst
- The ordered trajectory must match the developmental-stage arrangement shown in Fig. 1e
- The gene set entering DDRTree must be exactly the differentialGeneTest output at qval = 0.01
- DDRTree reduction must be run via monocle2 reduceDimension (not monocle3, which is what the TimeTalk README installs for its later steps)

## Limitations

- The TimeTalk repository installation instructions use monocle3 (cole-trapnell-lab/monocle3), so reproducing the monocle2 DDRTree trajectory requires installing monocle2 separately
- The trajectory is validated qualitatively against the Fig. 1e stage arrangement rather than a quantitative pseudotime metric

## Evidence

- [discussion] DDRTree trajectory step: "the differential expressed genes were used to perform DDRTree dimension reduction to reconstruct the development trajectory by the “reduceDimension” function."
- [discussion] DE gene cutoff: "we used the “differentialGeneTest” function to get differentially expressed genes (cutoff: qval = 0.01)"
- [discussion] Quantile normalization input: "Quantile normalization was performed with the function “normalize.quantiles” from R package preprocessCore (version 1.48.0)."
- [readme] monocle3 install in repo: "devtools::install_github('cole-trapnell-lab/monocle3')"
