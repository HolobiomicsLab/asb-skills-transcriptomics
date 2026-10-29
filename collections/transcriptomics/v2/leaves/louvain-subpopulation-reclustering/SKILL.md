---
name: louvain-subpopulation-reclustering
description: Use when a biologically annotated cell subpopulation needs finer subdivision,
  and a low-dimensional embedding from scGAE (20-D latent space of scRNA-seq counts)
  is available; specifically to confirm substructure such as ≥2 subclusters within
  Ba/Eo/MaP cells.
when_to_use_negative:
- When the goal is initial whole-dataset clustering rather than resolving substructure
  within an already-annotated population; run the full scGAE clustering step on all
  cells first.
- When competitive methods (e.g., DCA, scVI, PHATE) are substituted for scGAE — the
  article reports they did not identify the Ba/Eo/MaP subpopulations.
- When input data are already a 2D visualization rather than a high-dimensional preprocessed
  embedding; tSNE output is not suitable input for re-clustering.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: scGAE
  role: Produces the 20-D latent embedding in which Louvain clustering is performed;
    provides get_centers_louvain and model.clustering_train() for the clustering training
    phase
  repo: https://github.com/ZixiangLuo1161/scGAE
- name: t-SNE
  role: Visualizes the embedded data in two dimensions to inspect resolved subpopulations
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41598-021-99003-7
    title: A topology-preserving dimensionality reduction method for single-cell RNA-seq
      data using graph autoencoder
  source_content_hash: sha256:f7535b3d972f4314e233685387f1c7321da339245f8d0176b3c439a929fe59e1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ZixiangLuo1161/scGAE
  tools:
  - scGAE
  - t-SNE
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# louvain-subpopulation-reclustering

## Summary

Re-cluster a previously annotated cell population (e.g., the Ba/Eo/MaP hematopoietic progenitor subset from the HSPC dataset, NGDC HRA000084) in the latent space produced by scGAE's graph autoencoder, using Louvain centroids to resolve subpopulations that reflect differing differentiation potentials.

## When to use

Use when a biologically annotated cell subpopulation needs finer subdivision, and a low-dimensional embedding from scGAE (20-D latent space of scRNA-seq counts) is available; specifically to confirm substructure such as ≥2 subclusters within Ba/Eo/MaP cells.

## When NOT to use

- When the goal is initial whole-dataset clustering rather than resolving substructure within an already-annotated population; run the full scGAE clustering step on all cells first.
- When competitive methods (e.g., DCA, scVI, PHATE) are substituted for scGAE — the article reports they did not identify the Ba/Eo/MaP subpopulations.
- When input data are already a 2D visualization rather than a high-dimensional preprocessed embedding; tSNE output is not suitable input for re-clustering.

## Inputs

- Preprocessed scRNA-seq count matrix (scTransform-normalized, e.g., NGDC HRA000084 HSPC data)
- Cell-to-cell adjacency matrix from KNN graph (K=35 default)
- 20-dimensional scGAE latent embedding (h5ad/CSV)
- Table of previously annotated Ba/Eo/MaP (Basophil/Eosinophil/Mast progenitor) cell identities

## Outputs

- Subpopulation-label table assigning Ba/Eo/MaP cells to Louvain-derived subclusters
- Latent embedding saved as h5ad/CSV with subcluster annotations
- 2D tSNE visualization of embedded cells colored by subcluster (cf. Fig. 4a,b)

## How to apply

First preprocess the count matrix with scTransform in the Seurat package, then run the scGAE pipeline (Run_scGAE.ipynb) to embed cells into an intermediate 20-dimensional latent space, constructing the cell-to-cell KNN graph with the study's default K=35 (adjusted to the dataset). Filter to the cells of interest — in the HSPC analysis, the previously annotated Basophil/Eosinophil/Mast progenitor (Ba/Eo/MaP) cells. Use the pre-trained network's embedded data and obtain initial cluster centroids by Louvain clustering (via get_centers_louvain in the scGAE repo), then perform clustering training with model.clustering_train() to refine centroids. Verify by labeling cells with the resulting subcluster assignments and comparing against Fig. 4a,b: at least 2 subclusters should be resolved within Ba/Eo/MaP; competitive methods (DCA, scVI, scDeepCluster, PHATE, SAUCIE, scGNN, ZINB-WaVE, Ivis) should fail to resolve these subpopulations, supporting scGAE's higher statistical power to detect substructure.

## Related tools

- **scGAE** (Produces the 20-D latent embedding in which Louvain clustering is performed; provides get_centers_louvain and model.clustering_train() for the clustering training phase) — https://github.com/ZixiangLuo1161/scGAE
- **t-SNE** (Visualizes the embedded data in two dimensions to inspect resolved subpopulations)

## Examples

```
from scgae import SCGAE; centers = get_centers_louvain(model.embedding(), n_clusters=2); model.clustering_train(); sub_labels = louvain_predict(model.embedding(), centers)
```

## Evaluation signals

- At least 2 subclusters are resolved within the Ba/Eo/MaP population, matching the structure reported in Fig. 4a,b
- Subcluster assignments are obtained from Louvain initial centroids refined by model.clustering_train() on the pre-trained scGAE embedding
- Latent embedding is 20-dimensional (intermediate dimension) and exports cleanly as h5ad/CSV with subpopulation labels
- tSNE 2D plot of the embedding visually separates the resolved subpopulations
- Competitive methods (DCA, scVI, scDeepCluster, PHATE, SAUCIE, scGNN, ZINB-WaVE, Ivis) fail to resolve these subclusters on the same data

## Limitations

- No random seed or stochastic-variation control is reported, so subcluster results may vary between runs
- The optimal K for KNN graph construction varies among datasets and different parts of a dataset; K=35 is only the study default
- No software/package versions beyond the README requirements (e.g., spektral 0.6.1, tensorflow 2.3.0) are specified in the paper

## Evidence

- [discussion] Louvain centroids refine subclusters: "In experiments, we use the pre-trained network to generate the initial embedded data which are clustered to obtain the initial centroids by Louvain37."
- [discussion] Default K for graph construction: "The default K is predened as 35 in this study and adjusted according to the datasets in our experiments."
- [discussion] Intermediate embedding plus tSNE visualization: "we embedded the scRNA-seq data into an intermediate dimension and used tSNE to visualize the embedded data into a two-dimensional space."
- [readme] README clustering training procedure: "After getting the clustering centers using get_centers_louvain function, perform the clustering training using model.clustering_train() function."
- [discussion] Preprocessing with scTransform: "The scRNA-seq data preprocessing was conducted using scTransform40 in The Seurat package41."
