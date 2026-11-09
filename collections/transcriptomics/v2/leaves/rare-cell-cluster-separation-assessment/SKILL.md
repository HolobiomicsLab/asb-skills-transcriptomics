---
name: rare-cell-cluster-separation-assessment
description: Use when a scRNA-seq clustering comparison must determine whether any
  method can separate known rare subclusters, and when external metrics (ARI/NMI)
  and internal metrics (per-cell silhouette) should be reported against a Louvain
  resolution sweep.
when_to_use_negative:
- Input is real data without ground-truth labels — ARI/NMI cannot be computed and
  the comparison must rely on internal metrics or annotation only
- The goal is de novo rare-cell discovery in a single broad cluster (use SCISSORS
  reclustering directly) rather than benchmarking separation quality across methods
- The dataset has no rare or low-abundance subpopulations, where the silhouette/reclustering
  framework provides no advantage over standard Louvain
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: SCISSORS
  role: Iterative semi-supervised reclustering method under test; separates the two
    rare clusters and computes per-cell silhouette scores
  repo: https://github.com/jr-leary7/SCISSORS
- name: Seurat
  role: Provides the scRNA-seq infrastructure, preprocessing (NormalizeData, FindVariableFeatures,
    ScaleData, RunPCA) and Louvain clustering swept at resolutions 0.5–2.0
  repo: https://github.com/jr-leary7/SCISSORS
- name: cluster
  role: cluster::silhouette routine used to compute per-cell silhouette scores on
    PCA-space distances
- name: ggplot2
  role: Builds the violin-plot comparison of silhouette distributions and the resolution-sweep
    figures
- name: ggsignif
  role: Adds wilcox.test significance bars to the silhouette violin plots
- name: dplyr
  role: Tidy data manipulation in the R workflow
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad449
    title: <u>S</u>ub-<u>C</u>luster <u>I</u>dentification through <u>S</u>emi-<u>S</u>upervised
      <u>O</u>ptimization of <u>R</u>are-Cell <u>S</u>ilhouettes (SCISSORS) in single-cell
      RNA-sequencing
  source_content_hash: sha256:7b6db35c3acb592ba337ea4de0a9f13c9242a000db4bed042349e42633241e0d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jr-leary7/SCISSORS
  tools:
  - R
  - SCISSORS
  - Seurat
  - cluster
  - dplyr
  - ggplot2
  - ggsignif
  - magrittr
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# rare-cell-cluster-separation-assessment

## Summary

Benchmark clustering methods on a simulated scRNA-seq dataset containing rare, closely placed clusters by computing silhouette scores (cluster::silhouette in PCA space), ARI, and NMI against ground truth, and sweeping Seurat Louvain resolutions to show whether higher resolution improves separation.

## When to use

Use when a scRNA-seq clustering comparison must determine whether any method can separate known rare subclusters, and when external metrics (ARI/NMI) and internal metrics (per-cell silhouette) should be reported against a Louvain resolution sweep.

## When NOT to use

- Input is real data without ground-truth labels — ARI/NMI cannot be computed and the comparison must rely on internal metrics or annotation only
- The goal is de novo rare-cell discovery in a single broad cluster (use SCISSORS reclustering directly) rather than benchmarking separation quality across methods
- The dataset has no rare or low-abundance subpopulations, where the silhouette/reclustering framework provides no advantage over standard Louvain

## Inputs

- Simulated pancreas scRNA-seq count matrix (996 cells x 15,270 genes)
- Ground-truth cluster labels for the 7 clusters, including two rare closely placed clusters
- Seurat object after PrepareData/NormalizeData/FindVariableFeatures/ScaleData/RunPCA preprocessing
- PCA embeddings for silhouette distance computation

## Outputs

- Metrics table of ARI, NMI, and silhouette score per clustering method (saved as PDF)
- t-SNE panels of per-method cluster assignments on a shared embedding
- ggplot2 violin plot of per-method per-cell silhouette distributions with wilcox.test significance bars
- Louvain resolution-sweep figure showing ARI, NMI, and silhouette vs resolution (0.5–2.0)

## How to apply

Load the simulated dataset (996 cells, 15,270 genes) with its 7 ground-truth labels, preprocess via the SCISSORS/Seurat pipeline (NormalizeData, FindVariableFeatures with vst and 2000 features, ScaleData, RunPCA), then run the eight candidate methods including SCISSORS reclustering and Seurat Louvain at resolutions 0.5, 1.0, 1.5, and 2.0. Compute per-cell silhouette scores with cluster::silhouette on PCA-space distances and external ARI/NMI against the 7 true clusters. Visualize each method's labels on a shared t-SNE embedding and per-method silhouette distributions as ggplot2 violin plots with wilcox.test significance bars (ggsignif). Plot ARI, NMI, and silhouette as functions of Louvain resolution from 0.5 to 2.0; the rationale (per the SCISSORS paper) is that silhouette scoring estimates cluster heterogeneity and iterative semi-supervised reclustering reveals rare cells that fixed-resolution Louvain misses.

## Related tools

- **SCISSORS** (Iterative semi-supervised reclustering method under test; separates the two rare clusters and computes per-cell silhouette scores) — https://github.com/jr-leary7/SCISSORS
- **Seurat** (Provides the scRNA-seq infrastructure, preprocessing (NormalizeData, FindVariableFeatures, ScaleData, RunPCA) and Louvain clustering swept at resolutions 0.5–2.0) — https://github.com/jr-leary7/SCISSORS
- **cluster** (cluster::silhouette routine used to compute per-cell silhouette scores on PCA-space distances)
- **ggplot2** (Builds the violin-plot comparison of silhouette distributions and the resolution-sweep figures)
- **ggsignif** (Adds wilcox.test significance bars to the silhouette violin plots)
- **dplyr** (Tidy data manipulation in the R workflow)

## Examples

```
sils_SCISSORS <- ComputeSilhouetteScores(pbmc_SCISSORS, avg = FALSE)
```

## Evaluation signals

- SCISSORS attains ARI = 0.987, NMI = 0.985, and silhouette = 0.693 on the simulated pancreas dataset
- SCISSORS is the only one of the eight methods that separates the two rare clusters (check t-SNE panels for exactly 7 matching groups)
- The resolution sweep shows resolution 0.5 with the highest ARI/NMI/silhouette, so increasing resolution does not improve clustering
- Silhouette scores computed per cell via cluster::silhouette on PCA-space distances yield second-highest aggregate score for SCISSORS (0.693) with plausible per-cluster distributions in the violin plot
- All figures export as PDFs and the metrics table contains one row per candidate method (eight methods)

## Limitations

- Small differences in analysis and parameter choice can greatly alter scRNA-seq clustering results, so the benchmark is sensitive to preprocessing choices (e.g. 2000 HVGs, 10 PCs)
- Silhouette-based heterogeneity estimation assumes rare subclusters are spatially coherent in PCA space; closely placed rare clusters may still be merged by all methods
- The finding is demonstrated on a simulated 996-cell pancreas dataset; performance on real data with noisier ground truth is not established by this skill

## Evidence

- [abstract] silhouette scoring for heterogeneity estimation: "SCISSORS employs silhouette scoring for the estimation of heterogeneity of clusters and reveals rare cells in heterogenous clusters by a multi-step semi-supervised reclustering process"
- [intro] Seurat workflow extension: "This package enables researchers to efficiently, accurately, and reproducibly uncover rare celltypes and other small subgroups using an iterative reclustering optimization routine built as an"
- [methods] ComputeSilhouetteScores call: "sils_SCISSORS <- ComputeSilhouetteScores(pbmc_SCISSORS, avg = FALSE)"
- [methods] variable feature selection parameters: "FindVariableFeatures(selection.method = "vst", nfeatures = 2000, verbose = FALSE)"
- [readme] README PrepareData usage: "seu_pbmc <- SeuratData::LoadData("pbmc3k") %>% 
            PrepareData(n.HVG = 4000, 
                        n.PC = 15, 
                        which.dim.reduc = "umap","
