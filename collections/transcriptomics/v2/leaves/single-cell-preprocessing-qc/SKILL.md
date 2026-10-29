---
name: single-cell-preprocessing-qc
description: Use when raw CITE-seq count matrices (transcriptome + surface-protein/ADT)
  from 10x Genomics or similar must be cleaned and prepared — e.g. pbmc_1k_protein_v3-style
  datasets — before pairwise-distance fusion and Louvain clustering in LinQ-View.
when_to_use_negative:
- 'Massive datasets (e.g. ~170K cells with 228 ADT features): LinQ-View failed there
  while Seurat 4 succeeded; the article recommends Seurat 4 for massive CITE-seq data
  with many ADT features.'
- 'Datasets with large numbers of ADT features: the article states LinQ-View is best
  suited to large datasets with a limited number of ADT features (<50).'
- Input is already a preprocessed/normalized Seurat object with QC complete — re-applying
  these thresholds would discard cells redundantly.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: LinQ-View
  role: 'Primary toolkit: performs multimodal CITE-seq preprocessing, sigmoid-filtered
    ADT distances, L∞ fusion, and default Seurat Louvain clustering'
  repo: https://github.com/WilsonImmunologyLab/LinQView
- name: Seurat
  role: Backend for QC, normalization/scaling, Louvain community-detection clustering
    and (v3.9.9) batch correction
  repo: https://satijalab.org/seurat/
- name: ROGUE
  role: Cluster purity assessment (Ratio of Global Unshifted Entropy) to verify resulting
    clusters are pure cell populations
  repo: https://github.com/PaulingLiu/ROGUE
- name: R
  role: Execution environment (R v3.6.3 used in the paper) hosting LinQ-View, Seurat,
    dplyr, ggplot2, umap
  repo: https://www.r-project.org/
- name: ggplot2
  role: Rendering UMAP comparison figures (v3.3.2, e.g. the Figure 5 panel)
  repo: https://ggplot2.tidyverse.org/
- name: umap
  role: Non-linear dimension reduction for visualization (umap v0.2.6.0)
provenance:
  source_task_ids:
  - task_006
  - task_008
  source_papers:
  - doi: 10.1016/j.crmeth.2021.100056
    title: Improved integration of single-cell transcriptome and surface protein expression
      by LinQ-View
  source_content_hash: sha256:b2e7a84f72bd17144ae83223f9aded486d7fc88e0987d8c5c75a5287724e712e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WilsonImmunologyLab/LinQView
  tools:
  - LinQ-View
  - R
  - ROGUE
  - Seurat
  - dplyr
  - ggplot2
  - umap
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# single-cell-preprocessing-qc

## Summary

LinQ-View-style preprocessing and quality control for CITE-seq data: cell-level QC (gene and mitochondrial thresholds), normalization/scaling of both transcriptome and ADT modalities, HVG identification, and PCA ahead of joint multimodal clustering.

## When to use

Use when raw CITE-seq count matrices (transcriptome + surface-protein/ADT) from 10x Genomics or similar must be cleaned and prepared — e.g. pbmc_1k_protein_v3-style datasets — before pairwise-distance fusion and Louvain clustering in LinQ-View.

## When NOT to use

- Massive datasets (e.g. ~170K cells with 228 ADT features): LinQ-View failed there while Seurat 4 succeeded; the article recommends Seurat 4 for massive CITE-seq data with many ADT features.
- Datasets with large numbers of ADT features: the article states LinQ-View is best suited to large datasets with a limited number of ADT features (<50).
- Input is already a preprocessed/normalized Seurat object with QC complete — re-applying these thresholds would discard cells redundantly.

## Inputs

- 10x Genomics CITE-seq count matrices (e.g. pbmc_1k_protein_v3, pbmc_10k_protein_v3, malt_10k_protein_v3, 5k_pbmc_protein_v3)
- Transcriptome and ADT count matrices from Mendeley dataset 3jdywv5jrv/3 (COVID-19 patient CD19+ B cells)
- R statistical environment with LinQ-View v0.99, Seurat v3.9.9, dplyr v1.0.2

## Outputs

- QC-filtered, normalized Seurat object with transcriptome and ADT modalities scaled and HVGs identified
- PCA embeddings per modality feeding pairwise distance matrices and L∞ joint distance
- Cell-group/cluster assignments with UMAP coordinates exported (e.g. linqview_dataset6_bcell_clusters.csv)
- Comparison figures rendered with ggplot2 v3.3.2 (e.g. Figure 5 batch-correction panel, Figure 7-style B cell plots)

## How to apply

Load the transcriptome and ADT count matrices into R v3.6.3 via LinQ-View (LinQ-View v0.99, Seurat v3.9.9 backend). Apply cell quality control excluding cells with fewer than 200 or more than 2500 detected genes, then set a soft threshold on mitochondrial gene proportion at the 95th percentile of the dataset distribution, subject to a sealing point of 10% as the maximum threshold (optionally remove unwanted genes). Normalize and scale both the transcriptome and surface protein modalities, identify highly variable genes, and run PCA. Downstream, compute cell-cell pairwise distances per modality — applying the sigmoid filter to the ADT modality distance so no single surface marker dominates — and fuse with the L∞ norm before Seurat Louvain community-detection clustering and UMAP (umap v0.2.6.0) embedding. These exact thresholds and steps reproduce the paper's PBMC 1K + PBMC 10K batch-correction comparison and the SARS-CoV-2 B cell (Dataset 6, Mendeley 3jdywv5jrv/3) analyses.

## Related tools

- **LinQ-View** (Primary toolkit: performs multimodal CITE-seq preprocessing, sigmoid-filtered ADT distances, L∞ fusion, and default Seurat Louvain clustering) — https://github.com/WilsonImmunologyLab/LinQView
- **Seurat** (Backend for QC, normalization/scaling, Louvain community-detection clustering and (v3.9.9) batch correction) — https://satijalab.org/seurat/
- **ROGUE** (Cluster purity assessment (Ratio of Global Unshifted Entropy) to verify resulting clusters are pure cell populations) — https://github.com/PaulingLiu/ROGUE
- **R** (Execution environment (R v3.6.3 used in the paper) hosting LinQ-View, Seurat, dplyr, ggplot2, umap) — https://www.r-project.org/
- **ggplot2** (Rendering UMAP comparison figures (v3.3.2, e.g. the Figure 5 panel)) — https://ggplot2.tidyverse.org/
- **umap** (Non-linear dimension reduction for visualization (umap v0.2.6.0))

## Examples

```
install.packages("tidyverse"); devtools::install_github("PaulingLiu/ROGUE")
```

## Evaluation signals

- QC applied correctly: all cells with <200 or >2500 detected genes are excluded, and mitochondrial soft threshold sits at the 95th percentile with a 10% sealing maximum.
- Reproducing the PBMC 1K + PBMC 10K comparison yields 24 cell groups in the no-batch-correction control and 21 groups after Seurat 3 batch correction.
- On Dataset 6, derived clusters match reported Figure 7 assignments (RNA clusters 0/1 partitioning into LinQ-View clusters 0, 3, 4, 5; spike-specific memory-like clusters 6, 7, 9).
- Output CSV (e.g. linqview_dataset6_bcell_clusters.csv) exists with cluster, UMAP, and sample fields populated for all cells.
- ROGUE cluster purity scores confirm clusters are pure cell populations in both transcriptome and surface protein modalities.

## Limitations

- LinQ-View's pairwise distance computation scales quadratically with cell count, limiting time efficiency as dataset size increases.
- LinQ-View failed on a massive dataset of 170K cells with 228 ADT features where Seurat 4 succeeded.
- The pre-defined parameter K for the ROGUE score is technology-dependent and its value is not specified in the text.

## Evidence

- [discussion] QC gene-count cutoffs: "all cells that have less than 200 genes detected or more than 2500 genes detected have been excluded"
- [discussion] mitochondrial soft threshold: "A soft threshold was set to the 95th percentile of the current dataset distribution, and the soft threshold was subject to a sealing point as the maximum threshold"
- [discussion] preprocessing scope: "pre-processing steps include cell quality control, unwanted gene removal (optional), data normalization (for both transcriptome and surface protein data), data scaling"
- [discussion] ADT sigmoid filter: "we introduced a sigmoid filter into the distance calculation of the ADT modality"
- [readme] LinQ-View purpose (README): "Integrate multiple modalities of CITE-seq data for single cell populations identification"
