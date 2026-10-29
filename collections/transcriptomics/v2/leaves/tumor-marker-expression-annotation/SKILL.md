---
name: tumor-marker-expression-annotation
description: Use when a broad tumor/epithelial cluster in a Seurat scRNA-seq object
  shows low silhouette scores suggesting hidden heterogeneity, and rare malignant
  subclones or subtype identities (e.g., basal-like PDAC) must be resolved and annotated.
when_to_use_negative:
- The broad cluster already shows high silhouette scores across cells, indicating
  it is homogeneous and unlikely to contain hidden rare subclusters
- The input is not scRNA-seq count data in a Seurat workflow — CONICSmat alignment-based
  CNV modes and DECODER weights expect single-cell expression matrices
- Cell type labels are already fine-grained and no rare-cell subgroup is suspected,
  so unsupervised whole-dataset clustering suffices
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: SCISSORS
  role: Silhouette-scored, multi-step semi-supervised reclustering of heterogeneous
    broad tumor clusters to resolve rare subclusters; provides PrepareData, ComputeSilhouetteScores,
    CosineDist, and getClusters
  repo: https://github.com/jr-leary7/SCISSORS
- name: Seurat
  role: Underlying single-cell infrastructure wrapping the SCISSORS workflow (subset,
    NormalizeData, FindVariableFeatures, ScaleData, RunPCA, DimPlot)
- name: CONICSmat
  role: Detects aberrant copy-number variations from the count table of reclustered
    cells to confirm malignant identity of the rare basal-like subcluster
  repo: https://github.com/Neurosurgery-Brain-Tumor-Center-DiazLab/CONICS
- name: DECODER (decoderr)
  role: Computes basal-like pathway weights and bcRatio to quantify basal-like subtype
    identity of the recovered subcluster
  repo: https://github.com/laurapeng/decoderr
- name: cluster
  role: Supplies the silhouette statistic used by ComputeSilhouetteScores on cosine
    distances of PCA embeddings
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad449
    title: <u>S</u>ub-<u>C</u>luster <u>I</u>dentification through <u>S</u>emi-<u>S</u>upervised
      <u>O</u>ptimization of <u>R</u>are-Cell <u>S</u>ilhouettes (SCISSORS) in single-cell
      RNA-sequencing
  source_content_hash: sha256:5876d3eedb9005d60920f1b7519348aa8f52cff601326cd94a16166cde84cd67
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jr-leary7/SCISSORS
  tools:
  - CONICSmat
  - DECODER (decoderr)
  - R
  - SCISSORS
  - Seurat
  - cluster
  - dplyr
  - ggplot2
  - magrittr
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# tumor-marker-expression-annotation

## Summary

Applies SCISSORS silhouette-guided semi-supervised reclustering to a broadly-defined malignant/tumor cell cluster in scRNA-seq data (e.g., Elyada human PDAC epithelial Clusters 5 and 9), then annotates the recovered rare subcluster's tumor phenotype via copy-number aberration (CONICSmat), Moffitt basal-like/classical signature scoring (VAM), and DECODER basal-like pathway weights with GATA6 depletion checks.

## When to use

Use when a broad tumor/epithelial cluster in a Seurat scRNA-seq object shows low silhouette scores suggesting hidden heterogeneity, and rare malignant subclones or subtype identities (e.g., basal-like PDAC) must be resolved and annotated.

## When NOT to use

- The broad cluster already shows high silhouette scores across cells, indicating it is homogeneous and unlikely to contain hidden rare subclusters
- The input is not scRNA-seq count data in a Seurat workflow — CONICSmat alignment-based CNV modes and DECODER weights expect single-cell expression matrices
- Cell type labels are already fine-grained and no rare-cell subgroup is suspected, so unsupervised whole-dataset clustering suffices

## Inputs

- Seurat scRNA-seq object with raw counts and pre-existing broad cluster labels (e.g., Elyada et al. 2019 human PDAC epithelial clusters)
- Pre-defined tumor/epithelial cluster membership vector identifying the broad cluster(s) to recluster
- Marker gene signature sets (e.g., Moffitt basal-like and classical PDAC signatures) for VAM scoring
- DECODER basal-like pathway weight reference for bcRatio computation

## Outputs

- Reclustered Seurat object with rare subcluster labels and per-cell silhouette scores
- CONICSmat CNV aberration calls supporting malignant identity of the subcluster
- VAM module scores for Moffitt basal-like and classical signatures per cell
- DECODER basal-like pathway weights, bcRatio, and differential marker expression (e.g., GATA6 depletion)

## How to apply

Subset the target cells from the Seurat object (e.g., epithelial Clusters 5 and 9 in the Elyada PDAC dataset) and run PrepareData with n.HVG = 4000, n.PC = 15, initial.resolution = 0.4, and random.seed = 629 to build the PCA/UMAP basis. Compute per-cell silhouette scores on the cosine distance of PCA embeddings (ComputeSilhouetteScores with avg = FALSE, using CosineDist and cluster::silhouette) to confirm the merged population is heterogeneous enough to recluster. Apply the SCISSORS multi-step semi-supervised reclustering routine to resolve rare subclusters, then annotate tumor identity downstream: run CONICSmat to detect aberrant copy-number variations supporting malignant status, score Moffitt basal-like and classical signatures with VAM, and compute DECODER (decoderr) basal-like pathway weights and bcRatio; verify subtype-defining marker depletion such as GATA6 through differential expression. The rationale is that existing methods that fix an optimal cluster number miss extremely low-abundance cells, whereas silhouette scoring flags heterogeneity and iterative reclustering exposes subgroups as small as ~148 cells (0.59% of the dataset).

## Related tools

- **SCISSORS** (Silhouette-scored, multi-step semi-supervised reclustering of heterogeneous broad tumor clusters to resolve rare subclusters; provides PrepareData, ComputeSilhouetteScores, CosineDist, and getClusters) — https://github.com/jr-leary7/SCISSORS
- **Seurat** (Underlying single-cell infrastructure wrapping the SCISSORS workflow (subset, NormalizeData, FindVariableFeatures, ScaleData, RunPCA, DimPlot))
- **CONICSmat** (Detects aberrant copy-number variations from the count table of reclustered cells to confirm malignant identity of the rare basal-like subcluster) — https://github.com/Neurosurgery-Brain-Tumor-Center-DiazLab/CONICS
- **DECODER (decoderr)** (Computes basal-like pathway weights and bcRatio to quantify basal-like subtype identity of the recovered subcluster) — https://github.com/laurapeng/decoderr
- **cluster** (Supplies the silhouette statistic used by ComputeSilhouetteScores on cosine distances of PCA embeddings)

## Examples

```
seu_pbmc <- SeuratData::LoadData("pbmc3k") %>% PrepareData(n.HVG = 4000, n.PC = 15, which.dim.reduc = "umap", initial.resolution = 0.4, random.seed = 629); sil_score_df <- ComputeSilhouetteScores(seu_pbmc, avg = FALSE)
```

## Evaluation signals

- The recovered basal-like subcluster contains approximately 148 cells (~0.59% of the dataset), matching the known rare-abundance ground truth
- VAM scoring shows significant enrichment of Moffitt basal-like signature genes with absence of classical genes in the subcluster
- DECODER returns high basal-like pathway weights and a high bcRatio for the subcluster
- CONICSmat reports strong copy-number aberrations in the subcluster, supporting malignant identity
- GATA6 expression is depleted in the subcluster relative to classical PDAC cells, and per-cell silhouette distributions before reclustering showed low median scores for the merged epithelial clusters

## Limitations

- Small differences in analysis parameters and choices can greatly alter clustering results, so the subcluster outcome is sensitive to PrepareData settings (n.HVG = 4000, n.PC = 15, initial.resolution = 0.4, random.seed = 629)
- Extremely low-abundance cells contribute subtly to global metrics, so methods that fix an optimal cluster number cannot resolve them; silhouette scoring and reclustering are required, adding computational steps
- The SCISSORS reclustering is semi-supervised and iterative, requiring human judgment about which broad clusters are heterogeneous enough to recluster

## Evidence

- [abstract] Silhouette-guided reclustering rationale: "SCISSORS employs silhouette scoring for the estimation of heterogeneity of clusters and reveals rare cells in heterogenous clusters by a multi-step semi-supervised reclustering process"
- [abstract] Purpose of subcluster profiling: "we present a carefully designed framework, SCISSORS, which accurately profiles subclusters within broad cluster(s) for the identification of rare cell types in scRNA-seq data"
- [abstract] Why low-abundance subclusters are missed: "Existing methods largely focus on determining the optimal number of robust clusters, which can be problematic for identifying cells of extremely low abundance due to their subtle contributions toward"
- [readme] PrepareData parameters from README: "PrepareData(n.HVG = 4000,
                        n.PC = 15,
                        which.dim.reduc = "umap",
                        initial.resolution = 0.4,
                        use.parallel ="
- [methods] Silhouette computation per cell: "sils_SCISSORS <- ComputeSilhouetteScores(pbmc_SCISSORS, avg = FALSE)"
- [intro] Package goal statement: "This package enables researchers to efficiently, accurately, and reproducibly uncover rare celltypes and other small subgroups using an iterative reclustering optimization routine"
