---
name: robust-subpopulation-identification-across-replicates
description: Use when scRNA-seq count matrices from two or more independent 10X replicates
  (e.g., matrix.mtx/gene.tsv/barcode.tsv) must be resolved into high-resolution cell
  subpopulations with a shuffled-data false-discovery control, and reproducible clusters
  must be matched across replicates.
when_to_use_negative:
- Input is not raw 10X scRNA-seq count data (e.g., already clustered feature tables
  or bulk RNA-seq) — read.data expects 10X format and the pipeline performs its own
  quality filtering
- Only a single replicate with no plan to assess cross-replicate cluster reproducibility
  via correlation — the robustness/similarity step requires multiple independently
  clustered datasets
- Compute environment without slurm or at least 8G of memory when running parallel
  shuffled-data simulations
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: robustSingleCell
  role: Core R pipeline providing initialize.project, read.data, get.variable.genes,
    add.confounder.variables, PCA, cluster.analysis (Phenograph over shuffled data),
    summarize, get.cluster.names, assess.cluster.similarity, and get.robust.cluster.similarity
  repo: https://github.com/asmagen/robustSingleCell
- name: R
  role: Runtime language in which the pipeline is executed and dependencies (devtools,
    BiocManager, limma, biomaRt) are installed
- name: slurm
  role: Batch scheduler used for parallel shuffled-data simulations and PCA/clustering
    runs (local = F option)
- name: GEOquery
  role: Downloads the raw LCMV1/LCMV2 10X data from GEO via download_LCMV()
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.celrep.2019.10.131
    title: Single-Cell Profiling Defines Transcriptomic Signatures Specific to Tumor-Reactive
      versus Virus-Responsive CD4+ T Cells
  source_content_hash: sha256:d43f4ed5aad079114ba19576803407e1ed1c9a5574c3d9b48cb1ea268bf7537d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/asmagen/robustSingleCell
  tools:
  - GEOquery
  - R
  - robustSingleCell
  - slurm
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# robust-subpopulation-identification-across-replicates

## Summary

Identify robust cell subpopulations from scRNA-seq data with the robustSingleCell pipeline, controlling false discovery of clusters by repeating Phenograph clustering over shuffled data, then integrating replicate datasets (e.g., LCMV1/LCMV2) via cross-replicate cluster correlation similarity analysis.

## When to use

Use when scRNA-seq count matrices from two or more independent 10X replicates (e.g., matrix.mtx/gene.tsv/barcode.tsv) must be resolved into high-resolution cell subpopulations with a shuffled-data false-discovery control, and reproducible clusters must be matched across replicates.

## When NOT to use

- Input is not raw 10X scRNA-seq count data (e.g., already clustered feature tables or bulk RNA-seq) — read.data expects 10X format and the pipeline performs its own quality filtering
- Only a single replicate with no plan to assess cross-replicate cluster reproducibility via correlation — the robustness/similarity step requires multiple independently clustered datasets
- Compute environment without slurm or at least 8G of memory when running parallel shuffled-data simulations

## Inputs

- 10X genomics scRNA-seq files (matrix.mtx, gene.tsv, barcode.tsv) for each replicate (e.g., LCMV1, LCMV2 CD44+ T cell data from GEO)
- Curated marker-gene table mapping cell types (e.g., Tfh, Th1, Tcmp, Treg, Tmem, CD8, CD4, Cycle) to genes
- Analysis directory paths for data.path and work.path

## Outputs

- Per-replicate high-resolution cluster assignments with named subpopulations (set.cluster.names)
- Cluster modularity vs. shuffled-data modularity bar plot across tested KNN ratios (Clustering.modularity figure)
- tSNE plots colored by cluster assignment, violin plots of cell-cycle score, PCA loadings and heatmaps
- Cross-replicate cluster similarity/correlation results (assess.cluster.similarity similarity table, map, and filtered.similarity of robust clusters)
- Differential expression statistics from get.robust.markers

## How to apply

Initialize separate projects per replicate with initialize.project, then read 10X data with read.data (quality-filtered, optionally subsampled, e.g. 500 cells), identify highly variable genes with get.variable.genes, and compute confounder scores (ribosomal, mitochondrial, cell.cycle, and controlled.mean.score marker sets such as exhaustion markers Pdcd1/Cd244/Havcr2/Ctla4/Cd160/Lag3/Tigit/Cd96) via add.confounder.variables. Run PCA, which performs multiple simulations of shuffled data to choose the appropriate number of PCs, then cluster.analysis, which runs KNN ratio-based (Phenograph) clustering over a range of resolutions and repeats it on shuffled data; select the smallest KNN ratio whose original-vs-shuffled modularity fold change (mdlrty/mean.shfl) exceeds 2 (0.05 in the LCMV example). Summarize with differential expression and tSNE, name clusters with get.cluster.names against a curated marker table (e.g., Tfh: Tcf7/Cxcr5/Bcl6; Th1: Cxcr6/Ifng/Tbx21) at min.fold=1.0, max.Qval=0.01. Finally, pool replicates via read.preclustered.datasets and assess.cluster.similarity to correlate cluster fold-change vectors across replicates, retaining reproducible subpopulations with get.robust.cluster.similarity (min.sd=qnorm(.9), max.q.val=0.01).

## Related tools

- **robustSingleCell** (Core R pipeline providing initialize.project, read.data, get.variable.genes, add.confounder.variables, PCA, cluster.analysis (Phenograph over shuffled data), summarize, get.cluster.names, assess.cluster.similarity, and get.robust.cluster.similarity) — https://github.com/asmagen/robustSingleCell
- **R** (Runtime language in which the pipeline is executed and dependencies (devtools, BiocManager, limma, biomaRt) are installed)
- **slurm** (Batch scheduler used for parallel shuffled-data simulations and PCA/clustering runs (local = F option))
- **GEOquery** (Downloads the raw LCMV1/LCMV2 10X data from GEO via download_LCMV())

## Examples

```
devtools::install_github("asmagen/robustsinglecell"); LCMV1 <- initialize.project(datasets = "LCMV1", origins = "CD44+ cells", experiments = "Rep1", data.path = file.path(tempdir(), "LCMV"), work.path = file.path(tempdir(), "LCMV/LCMV_analysis")); LCMV1 <- read.data(LCMV1, subsample = 500); LCMV1 <- cluster.analysis(LCMV1, local = T)
```

## Evaluation signals

- Chosen clustering resolution is the smallest tested KNN ratio with modularity fold change of original vs. shuffled analysis greater than 2 (mdlrty/mean.shfl > 2)
- Robust cross-replicate cluster similarities pass min.sd = qnorm(.9) and max.q.val = 0.01 in get.robust.cluster.similarity
- Cluster names assigned only where marker fold change ≥ 1.0 and Q value ≤ 0.01 (min.fold = 1.0, max.Qval = 0.01)
- Each replicate dataset is clustered separately before pooled integration (LCMV1 and LCMV2 analyzed individually, then read.preclustered.datasets)
- Post-filtering QC figures show mitochondrial score vs. UMI counts after read.data quality filtering, comparable to pre-filtering plots

## Limitations

- The robustSingleCell package is not currently maintained — the README asks interested users to contact the authors for collaboration
- The pipeline currently supports only slurm for parallel batch jobs, and requires at least 8G of memory
- When using your own data, 10X files must be placed in the data.path working directory; the genome directory depends on the alignment genome (e.g., outs/filtered_gene_bc_matrices/mm10)

## Evidence

- [intro] robustSingleCell is a pipeline designed to identify robust cell subpopulations using scRNAseq data: "robustSingleCell is a pipeline designed to identify robust cell subpopulations using scRNAseq data"
- [readme] The analysis is repeated multiple times over shuffled data to estimate the appropriate clustering resolution and control for false discovery of clusters.: "The analysis is repeated multiple times over shuffled data
to estimate the appropriate clustering resolution and control for false
discovery of clusters."
- [readme] We choose 0.05 for our KNN ratio, which is the smallest value tested with `mdlrty/mean.shfl` > 2.: "We choose 0.05
for our KNN ratio, which is the smallest value tested with
`mdlrty/mean.shfl` \> 2."
- [readme] The `PCA` function performs multiple simulation analyses of shuffled data to determine the appropriate number of PCs.: "The `PCA` function performs multiple simulation analyses of shuffled
data to determine the appropriate number of PCs."
- [methods] The `read.data` function reads the data in 10X genomics format and performs quality filtering as described in *Magen et al 2019*: "`read.data` function reads the data in 10X genomics format and performs
quality filtering as described in *Magen et al 2019*"
- [other] This package is not maintained at the moment -- please contact if you're interested in collaborating and improving this project.: "**This package is not maintained at the moment -- please contact if you're interested in collaborating and improving this project.**"
