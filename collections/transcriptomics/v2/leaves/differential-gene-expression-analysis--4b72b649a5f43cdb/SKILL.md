---
name: differential-gene-expression-analysis
description: Use when scRNAseq clusters (e.g., from LCMV1/LCMV2 CD44+ T cell 10X data)
  already exist and you need genes defining each cluster versus all others, plus subtype
  label assignment.
when_to_use_negative:
- Clusters have not yet been defined — the skill assumes cluster.analysis has already
  produced cluster assignments
- The goal is cross-dataset integration or cluster-similarity assessment (use assess.cluster.similarity/get.robust.cluster.similarity
  on the pooled environment instead)
- Input data are bulk RNA-seq or already-aggregated feature tables rather than per-cell
  10X count matrices
edam_operation: http://edamontology.org/operation_3562
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: robustSingleCell
  role: Pipeline whose summarize function performs limma-based one-vs-rest cluster
    DE; get.cluster.names and set.cluster.names assign subtype labels from marker
    genes
  repo: github.com/asmagen/robustSingleCell
- name: limma
  role: Underlying differential expression engine used by summarize for per-cluster
    one-vs-rest testing
- name: R
  role: Execution environment; robustSingleCell installed via devtools::install_github
    or CRAN
  repo: github.com/asmagen/robustSingleCell
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.celrep.2019.10.131
    title: Single-Cell Profiling Defines Transcriptomic Signatures Specific to Tumor-Reactive
      versus Virus-Responsive CD4+ T Cells
  source_content_hash: sha256:b834635bfe8ff572e725e6d8e557cefd9e1dcec0aab443c33a3b01ff996086a4
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - limma
  - robustSingleCell
  edam_operation: http://edamontology.org/operation_3562
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# differential-gene-expression-analysis

## Summary

Identify cluster-defining differentially expressed genes in single-cell RNAseq clusters using limma-based one-vs-rest differential expression via the robustSingleCell summarize function, then annotate clusters against curated T-cell subtype marker sets (Tfh, Th1, Tcmp, Treg, Tmem, CD8, CD4, Cycle).

## When to use

Use when scRNAseq clusters (e.g., from LCMV1/LCMV2 CD44+ T cell 10X data) already exist and you need genes defining each cluster versus all others, plus subtype label assignment.

## When NOT to use

- Clusters have not yet been defined — the skill assumes cluster.analysis has already produced cluster assignments
- The goal is cross-dataset integration or cluster-similarity assessment (use assess.cluster.similarity/get.robust.cluster.similarity on the pooled environment instead)
- Input data are bulk RNA-seq or already-aggregated feature tables rather than per-cell 10X count matrices

## Inputs

- Processed robustSingleCell project environments (e.g., LCMV1, LCMV2) after read.data, get.variable.genes, PCA and cluster.analysis
- 10X genomics-format input files (matrix.mtx, gene.tsv, barcode.tsv) downloaded via download_LCMV from GEO
- Curated marker-gene types dataframe (type/gene pairs for Tfh, Th1, Tcmp, Treg, Tmem, CD8, CD4, Cycle)

## Outputs

- Per-cluster limma differential expression statistics (gene, fold change, Q-value) written to the analysis folder by summarize
- Cluster-defining gene list passing FC > 1.25 and Q < 0.1 in both replicates (cluster_defining_genes.tsv)
- Annotated cluster names assigned via get.cluster.names/set.cluster.names and re-visualized by summarize (tSNE, marker heatmaps)

## How to apply

For each replicate project (LCMV1, LCMV2), run robustSingleCell's summarize function, which performs limma-based differential expression of every cluster against all remaining clusters and writes results to the analysis folder. Extract per-cluster statistics and retain cluster-defining genes passing fold change > 1.25 and Q-value < 0.1 (Benjamini–Hochberg FDR) independently in both biological replicates. Then call get.cluster.names with a curated marker-gene types dataframe — Tfh: Tcf7/Cxcr5/Bcl6; Th1: Cxcr6/Ifng/Tbx21; Tcmp: Ccr7/Bcl2/Tcf7; Treg: Foxp3/Il2ra; Tmem: Il7r/Ccr7; CD8: Cd8a; CD4: Cd4; Cycle: Mki67/Top2a/Birc5 — using min.fold = 1.0 and max.Qval = 0.01, and attach labels with set.cluster.names followed by a second summarize run. Correctness is judged by verifying the per-cluster gene/fold-change/Q-value output exists and that assigned labels are supported by the marker sets within the stated thresholds.

## Related tools

- **robustSingleCell** (Pipeline whose summarize function performs limma-based one-vs-rest cluster DE; get.cluster.names and set.cluster.names assign subtype labels from marker genes) — github.com/asmagen/robustSingleCell
- **limma** (Underlying differential expression engine used by summarize for per-cluster one-vs-rest testing)
- **R** (Execution environment; robustSingleCell installed via devtools::install_github or CRAN) — github.com/asmagen/robustSingleCell

## Examples

```
summarize(LCMV1, local = T); LCMV1_cluster_names <- get.cluster.names(LCMV1, types, min.fold = 1.0, max.Qval = 0.01); LCMV1 <- set.cluster.names(LCMV1, names = LCMV1_cluster_names)
```

## Evaluation signals

- cluster_defining_genes.tsv exists and contains one entry per annotated cluster with gene, fold-change and Q-value fields
- Retained genes pass fold change > 1.25 and Benjamini–Hochberg Q-value < 0.1 independently in both replicates (LCMV1 and LCMV2)
- Assigned cluster labels match one of the curated T-cell subtype marker sets within min.fold = 1.0 and max.Qval = 0.01
- A second summarize run after set.cluster.names regenerates figures (tSNE, marker heatmaps) showing the assigned labels
- Each annotated cluster's marker genes (e.g., Tcf7/Cxcr5/Bcl6 for Tfh) show significant enrichment in that cluster versus the rest

## Limitations

- The robustSingleCell package is not currently maintained, per the README
- Marker-gene types must be adapted by users to their own data; the default Tfh/Th1/Tcmp/Treg/Tmem/CD8/CD4/Cycle sets suit CD44+ T cell data
- The pipeline supports slurm for parallel batch jobs; local runs need adequate memory (tutorial requests 8G)
- Tutorial datasets are randomly downsampled to 500 cells, which may affect DE power relative to full datasets

## Evidence

- [readme] The summarize function which performs differential expression analysis, computes tSNE and visualizes the results in the analysis folder.: "The `summarize` function which performs differential expression
analysis, computes tSNE and visualizes the results in the analysis
folder."
- [readme] get.cluster.names assigns cluster names using a customized marker set: "After differential expression analysis, `get.cluster.names`
assigns clusters with names using a customized set of marker genes which
users should adapt to their own data."
- [full_text] Cluster-defining genes via Limma with FC > 1.25 and Q < 0.1 in two replicates: [evidence span withheld — anchor status: not_found]
- [full_text] Cluster DE relative to all other clusters within the same dataset: [evidence span withheld — anchor status: not_found]
- [intro] robustSingleCell identifies robust cell subpopulations using scRNAseq data: "robustSingleCell is a pipeline designed to identify robust cell subpopulations using scRNAseq data"
