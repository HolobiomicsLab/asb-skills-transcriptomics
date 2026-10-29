---
name: gene-expression-contour-overlay-on-tsne
description: Use when a pooled t-SNE embedding of scRNAseq clusters needs annotation
  with expression contours of marker genes (e.g. Cd4, Cd8a) to identify compartment
  or lineage identity of clusters.
when_to_use_negative:
- Datasets have not been individually preclustered — the pooled t-SNE relies on read.preclustered.datasets;
  run the per-dataset pipeline (read.data, get.variable.genes, PCA, cluster.analysis)
  first.
- You need quantitative differential-expression statistics per cluster rather than
  spatial visualization — use get.robust.markers / summarize differential expression
  instead.
- The goal is trajectory (pseudotime) ordering of TIL subpopulations — use Monocle
  v2.9.0 DDRTree instead of a static t-SNE contour overlay.
edam_operation: http://edamontology.org/operation_0570
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: robustSingleCell
  role: Provides the pooled t-SNE computation (summarize / run_tSNE) and the plot_contour_overlay_tSNE
    function that draws normalized expression contours on the t-SNE
  repo: github.com/asmagen/robustSingleCell
- name: R
  role: Runtime environment in which the robustSingleCell pipeline and plotting functions
    are executed
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.celrep.2019.10.131
    title: Single-Cell Profiling Defines Transcriptomic Signatures Specific to Tumor-Reactive
      versus Virus-Responsive CD4+ T Cells
  source_content_hash: sha256:614c51111adfc0198c009e6eff2efb47d68b30433488e424d1df0fc55b8394d6
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - robustSingleCell
  edam_operation: http://edamontology.org/operation_0570
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# gene-expression-contour-overlay-on-tsne

## Summary

A visualization skill that overlays normalized gene-expression contours onto a pooled 2D t-SNE embedding using robustSingleCell's plot_contour_overlay_tSNE, producing flow-cytometry-like contour figures (e.g. CD4.contour.tSNE.png, CD8.contour.tSNE.png) that annotate cell states by marker expression.

## When to use

Use when a pooled t-SNE embedding of scRNAseq clusters needs annotation with expression contours of marker genes (e.g. Cd4, Cd8a) to identify compartment or lineage identity of clusters.

## When NOT to use

- Datasets have not been individually preclustered — the pooled t-SNE relies on read.preclustered.datasets; run the per-dataset pipeline (read.data, get.variable.genes, PCA, cluster.analysis) first.
- You need quantitative differential-expression statistics per cluster rather than spatial visualization — use get.robust.markers / summarize differential expression instead.
- The goal is trajectory (pseudotime) ordering of TIL subpopulations — use Monocle v2.9.0 DDRTree instead of a static t-SNE contour overlay.

## Inputs

- robustSingleCell pooled analysis environment built from independently preclustered replicate datasets (via initialize.project + read.preclustered.datasets)
- 10X genomics scRNAseq count data (matrix.mtx, gene.tsv, barcode.tsv) underlying the preclustered replicates
- Selected marker genes for contour overlay (e.g. Cd4, Cd8a)

## Outputs

- tsne.png — pooled t-SNE plot colored by cluster assignment
- CD4.contour.tSNE.png — flow-cytometry-like contour overlay of Cd4 expression on t-SNE
- CD8.contour.tSNE.png — flow-cytometry-like contour overlay of Cd8a expression on t-SNE

## How to apply

First generate the pooled t-SNE: initialize a pooled project over the independently preclustered replicate datasets with initialize.project and read.preclustered.datasets, add confounder variables (ribosomal.score, mitochondrial.score, cell.cycle.score, Exhaustion via controlled.mean.score), then re-run PCA with clear.previously.calculated.clustering = F and compute the 2D t-SNE via the summarize / run_tSNE step, saving tsne.png colored by cluster assignment. Then call plot_contour_overlay_tSNE on the pooled environment for each selected marker (e.g. Cd4, Cd8a), drawing contour overlays of normalized expression on the same t-SNE coordinates. Read the contours like flow-cytometry plots: high-expression contour islands localize marker-positive subpopulations within the embedding. The resulting figures (CD4.contour.tSNE.png, CD8.contour.tSNE.png) are written to the analysis figs folder alongside tsne.png.

## Related tools

- **robustSingleCell** (Provides the pooled t-SNE computation (summarize / run_tSNE) and the plot_contour_overlay_tSNE function that draws normalized expression contours on the t-SNE) — github.com/asmagen/robustSingleCell
- **R** (Runtime environment in which the robustSingleCell pipeline and plotting functions are executed)

## Examples

```
pooled_env <- initialize.project(datasets = c("LCMV1","LCMV2"), origins = c("CD44+ cells","CD44+ cells"), experiments = c("Rep1","Rep2"), data.path = file.path(tempdir(),"LCMV"), work.path = file.path(tempdir(),"LCMV/LCMV_analysis")); pooled_env <- read.preclustered.datasets(pooled_env); summarize(pooled_env, contrast = "datasets", local = T)
```

## Evaluation signals

- tsne.png, CD4.contour.tSNE.png, and CD8.contour.tSNE.png are all present in the analysis figs folder after the run.
- Contour overlays are drawn on the same t-SNE coordinates as tsne.png, so contour islands align with visible cluster assignments.
- CD4 and CD8a contours show mutually exclusive high-expression regions, consistent with CD4+ vs CD8+ T cell identity in the Ciucci et al. 2019 CD44+ T cell data.
- Pooled embedding shows little transcriptomic overlap between TILs and dLN cells, supporting distinctness of the tumor-specific response.
- PCA on the pooled environment was re-run with clear.previously.calculated.clustering = F so per-dataset cluster assignments are preserved in the pooled t-SNE.

## Limitations

- The robustSingleCell package is not currently maintained, per the README.
- The t-SNE embedding itself does not quantify overlap; in the source study TILs, dLN cells, and Arm cells showed little transcriptomic overlap, so contours may not reveal shared states between compartments.
- The pipeline supports slurm for parallel batch jobs and the example analysis requires at least 8G of memory.

## Evidence

- [other] Annotate the t-SNE with contour overlays of normalized expression of selected genes (e.g. Cd4, Cd8a) using plot_contour_overlay_tSNE: [evidence span withheld — anchor status: not_found]
- [readme] Figure 7 shows the tSNE visualization of the cells, color coded by cluster assignment.: "Figure 7 shows the tSNE visualization of the cells, color coded by
cluster assignment."
- [readme] The summarize function which performs differential expression analysis, computes tSNE and visualizes the results in the analysis folder.: "The `summarize` function which performs differential expression
analysis, computes tSNE and visualizes the results in the analysis
folder."
- [intro] robustSingleCell is a pipeline designed to identify robust cell subpopulations using scRNAseq data: "robustSingleCell is a pipeline designed to identify robust cell
subpopulations using scRNAseq data"
- [results] The pooled t-SNE embedding shows that TILs, dLN cells, and Arm cells have little transcriptomic overlap: [evidence span withheld — anchor status: not_found]
