---
name: graph-autoencoder-embedding
description: Use when scRNA-seq count matrices must be embedded for visualization,
  clustering, or trajectory inference while preserving inter-cell topological structure
  (e.g., keeping transcriptionally similar populations close), and deep-learning competitors
  lose cell-cell relationships.
when_to_use_negative:
- When a direct 2D embedding is required without an intermediate step — the paper
  notes 'A better 2D visualization method is needed to preserve the topological structure'
  and uses intermediate-dimension-plus-tSNE instead.
- When input is already a feature/embedding table rather than a raw count matrix requiring
  scTransform preprocessing and KNN graph construction.
- When the optimal KNN graph connectivity must be fixed a priori — the article states
  'the optimal K varies among different datasets and different parts of a dataset'.
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scGAE
  role: multitask-oriented graph autoencoder that builds a cell graph and embeds cells
    into a 20-dimensional latent space preserving topological and feature information
  repo: github.com/ZixiangLuo1161/scGAE
- name: t-SNE
  role: 2D visualization of the 20-dimensional scGAE embedding
- name: Spektral
  role: Python package used to implement the scGAE model (spektral 0.6.1 in README
    requirements)
- name: scVI
  role: deep-learning competitor embedding method for NMI and Kendall-tau benchmarks
- name: DCA
  role: competitor embedding method applied with default settings on simulated and
    empirical datasets
- name: PHATE
  role: competitor dimensionality-reduction baseline for clustering and trajectory
    benchmarks
- name: SAUCIE
  role: competitor embedding method in the benchmark comparisons
- name: scGNN
  role: graph-based competitor with similar trajectory-inference performance but slower
    running time than scGAE
- name: ZINB-WaVE
  role: competitor embedding method in the benchmark comparisons
- name: Ivis
  role: competitor embedding method in the benchmark comparisons
- name: scDeepCluster
  role: competitor included in clustering NMI benchmarks (excluded from trajectory
    Kendall-tau comparison as in Fig. 3a,b)
provenance:
  source_task_ids:
  - task_007
  - task_008
  - task_004
  - task_005
  source_papers:
  - doi: 10.1038/s41598-021-99003-7
    title: A topology-preserving dimensionality reduction method for single-cell RNA-seq
      data using graph autoencoder
  source_content_hash: sha256:fb5c4167285d325832474d8dfb7bdf27c2b6a5a1a92ab907eaaf79f086d4f88c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DCA
  - Ivis
  - PHATE
  - SAUCIE
  - Spektral
  - ZINB-WaVE
  - scDeepCluster
  - scGAE
  - scGNN
  - scVI
  - t-SNE
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8947368421052632
n_claims: 19
---

# graph-autoencoder-embedding

## Summary

A skill for producing topology-preserving low-dimensional embeddings of scRNA-seq data with scGAE: build a KNN cell graph (default K=35), preprocess with scTransform, and train a multitask-oriented graph autoencoder (Spektral/TensorFlow) to embed cells into a 20-dimensional latent space that preserves both topological structure and feature information.

## When to use

Use when scRNA-seq count matrices must be embedded for visualization, clustering, or trajectory inference while preserving inter-cell topological structure (e.g., keeping transcriptionally similar populations close), and deep-learning competitors lose cell-cell relationships.

## When NOT to use

- When a direct 2D embedding is required without an intermediate step — the paper notes 'A better 2D visualization method is needed to preserve the topological structure' and uses intermediate-dimension-plus-tSNE instead.
- When input is already a feature/embedding table rather than a raw count matrix requiring scTransform preprocessing and KNN graph construction.
- When the optimal KNN graph connectivity must be fixed a priori — the article states 'the optimal K varies among different datasets and different parts of a dataset'.

## Inputs

- preprocessed scRNA-seq count matrix (e.g., H5AD/CSV) from datasets such as NGDC HRA000084 (HSPCs) or GEO GSE84133 (Baron human pancreas)
- ground-truth cluster labels or previously annotated cell-population labels (e.g., Ba/Eo/MaP progenitor cells) for subclustering or NMI evaluation
- Splatter-simulated datasets (e.g., scenario1–4) with simulated trajectory orderings or cluster labels

## Outputs

- 20-dimensional latent embedding of cells (e.g., embedding_coordinates.csv or h5ad)
- 2D tSNE visualization of the embedded data
- subpopulation-label table from Louvain re-clustering (e.g., ≥2 subclusters within Ba/Eo/MaP)
- pairwise population centroid-distance table (e.g., centroid_distances.csv)
- benchmark tables such as nmi_comparison.csv or kendall_tau_comparison.csv

## How to apply

Preprocess the count matrix using scTransform in the Seurat package (or scanpy in Python), then compute a cell-to-cell adjacency matrix with the KNN algorithm via the get_adj function, using the default K=35 adjusted per dataset. Build the SCGAE model from the repository modules (preprocessing.py, scgae.py, layers.py, losses.py, utils.py) and train it with model.train(); optionally obtain initial centroids with get_centers_louvain and run model.clustering_train(). Generate the latent embedding with model.embedding() into a 20-dimensional intermediate space; for 2D visualization apply tSNE on the embedding, since the graph autoencoder does not directly embed into two dimensions. Downstream, apply Louvain clustering on the embedding for substructure detection (e.g., resolving subpopulations within Ba/Eo/MaP HSPC progenitors from NGDC HRA000084) or diffusion pseudotime for trajectory recovery, and compare centroid Euclidean distances among populations (e.g., the 14 Baron pancreas populations from GSE84133) to verify topological preservation.

## Related tools

- **scGAE** (multitask-oriented graph autoencoder that builds a cell graph and embeds cells into a 20-dimensional latent space preserving topological and feature information) — github.com/ZixiangLuo1161/scGAE
- **t-SNE** (2D visualization of the 20-dimensional scGAE embedding)
- **Spektral** (Python package used to implement the scGAE model (spektral 0.6.1 in README requirements))
- **scVI** (deep-learning competitor embedding method for NMI and Kendall-tau benchmarks)
- **DCA** (competitor embedding method applied with default settings on simulated and empirical datasets)
- **PHATE** (competitor dimensionality-reduction baseline for clustering and trajectory benchmarks)
- **SAUCIE** (competitor embedding method in the benchmark comparisons)
- **scGNN** (graph-based competitor with similar trajectory-inference performance but slower running time than scGAE)
- **ZINB-WaVE** (competitor embedding method in the benchmark comparisons)
- **Ivis** (competitor embedding method in the benchmark comparisons)
- **scDeepCluster** (competitor included in clustering NMI benchmarks (excluded from trajectory Kendall-tau comparison as in Fig. 3a,b))

## Examples

```
from scgae import SCGAE; from utils import get_adj; adj = get_adj(X, k=35); model = SCGAE(adj, X); model.train(); emb = model.embedding()
```

## Evaluation signals

- On Splatter scenario3/scenario4 clustering tasks, Louvain clustering on the scGAE embedding should yield the maximum NMI among the ten evaluated methods (scGAE, scDeepCluster, DCA, scVI, Ivis, SAUCIE, scGNN, ZINB-WaVE, PCA, PHATE), matching Fig. 3c,d.
- Diffusion pseudotime inferred on the scGAE embedding should achieve Kendall-tau correlation within [-1, 1] competitive with scGNN and scVI on scenario1 and scenario2, matching the relative ordering in Fig. 3a,b.
- On the Baron pancreas embedding, centroid Euclidean distances in the 20-D space should show the activated-quiescent stellate pair and the two ductal subtypes closer than unrelated population pairs (Fig. 4c,d pattern in centroid_distances.csv).
- Louvain re-clustering of previously annotated Ba/Eo/MaP cells on the scGAE embedding should resolve at least 2 subclusters, matching Fig. 4a,b structure.
- Pipeline completeness checks: embedding saved as h5ad/CSV, adjacency computed with K=35 (or justified adjustment), and scTransform preprocessing applied before model.training.

## Limitations

- No software/package versions were specified in the article itself (the README lists them: tensorflow 2.3.0, spektral 0.6.1, scanpy 1.8.0, etc.), and no random seed or stochastic-variation control is reported, so runs may vary.
- The optimal K for KNN graph construction remains unresolved and varies among datasets and dataset regions; the default K=35 must be adjusted per dataset.
- A better 2D visualization method than tSNE is still needed; the graph autoencoder cannot directly embed into two dimensions, requiring an intermediate 20-D embedding plus tSNE.
- scGAE and scGNN have similar trajectory-inference performance; scGAE's advantage is mainly in clustering quality and runtime (much shorter running time and slightly lower memory cost than scGNN).

## Evidence

- [abstract] scGAE builds a cell graph and uses a multitask-oriented graph autoencoder to preserve topological structure information and feature information: "scGAE builds a cell graph and uses a multitask-oriented graph autoencoder to preserve topological structure information and feature information"
- [discussion] we embedded the scRNA-seq data into an intermediate dimension and used tSNE to visualize the embedded data into a two-dimensional space.: "we embedded the scRNA-seq data into an intermediate dimension and used tSNE to visualize the embedded data into a two-dimensional space."
- [discussion] The default K is predened as 35 in this study and adjusted according to the datasets in our experiments.: "The default K is predened as 35 in this study and adjusted according to the datasets in our experiments."
- [discussion] The scRNA-seq data preprocessing was conducted using scTransform40 in The Seurat package41.: "The scRNA-seq data preprocessing was conducted using scTransform40 in The Seurat package41."
- [readme] Compute the adjacency matrix using get_adj function and build the SCGAE model.: "Compute the adjacency matrix using get_adj function and build the SCGAE model."
- [readme] Get the latent embedding of the data by model.embedding(): "Get the latent embedding of the data by model.embedding()"
- [discussion] We used the Python package Spektral32 to implement our model.: "We used the Python package Spektral32 to implement our model."
