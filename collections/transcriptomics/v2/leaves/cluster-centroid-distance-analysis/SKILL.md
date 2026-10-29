---
name: cluster-centroid-distance-analysis
description: Use when you have an embedded single-cell dataset (e.g., a 20-dim scGAE
  latent representation) with known cell-type/population labels and need to verify
  that known transcriptionally related populations (e.g., activated vs quiescent stellate
  cells, ductal subtypes) remain at short centroid distances.
when_to_use_negative:
- When the goal is only 2D visual inspection of clusters — scGAE embeds to an intermediate
  dimension and uses tSNE for 2D plots because 'the complex data structure is hard
  to be directly embedded into two-dimensional space by graph autoencoder', so 2D
  coordinates should not be used for centroid-distance quantification
- When no reliable population/cluster labels are available — centroids require assigning
  each cell to a population (e.g., Louvain-derived centers or the 14 known Baron populations)
- When the embedding method does not claim topology preservation (e.g., PCA/tSNE alone)
  — the short-distance relationships this analysis checks were demonstrated for scGAE,
  and competing methods projected ductal subtypes far apart
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scGAE
  role: graph autoencoder that embeds cells into the 20-dimensional latent space whose
    centroid distances are analyzed; implemented with the Python package Spektral
  repo: github.com/ZixiangLuo1161/scGAE
- name: Seurat (scTransform)
  role: preprocessing of the count matrix (e.g., Baron GSE84133) before KNN graph
    construction and embedding
- name: tSNE
  role: competitor/baseline method whose inter-population distances (e.g., ductal
    subtypes projected far apart) are compared against the scGAE centroid-distance
    pattern
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41598-021-99003-7
    title: A topology-preserving dimensionality reduction method for single-cell RNA-seq
      data using graph autoencoder
  source_content_hash: sha256:8840e8d047dc6fe8d5b8ec991e2c8e635e7696973f149c5fa837d0a93f95663d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Seurat (scTransform)
  - scGAE
  - tSNE
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# cluster-centroid-distance-analysis

## Summary

Quantify inter-population topological structure preserved by a low-dimensional embedding by computing per-cluster centroids (e.g., the 14 Baron pancreatic cell populations in a 20-dimensional scGAE latent space) and all pairwise Euclidean centroid distances, then checking that transcriptionally similar populations sit at short distances.

## When to use

Use when you have an embedded single-cell dataset (e.g., a 20-dim scGAE latent representation) with known cell-type/population labels and need to verify that known transcriptionally related populations (e.g., activated vs quiescent stellate cells, ductal subtypes) remain at short centroid distances.

## When NOT to use

- When the goal is only 2D visual inspection of clusters — scGAE embeds to an intermediate dimension and uses tSNE for 2D plots because 'the complex data structure is hard to be directly embedded into two-dimensional space by graph autoencoder', so 2D coordinates should not be used for centroid-distance quantification
- When no reliable population/cluster labels are available — centroids require assigning each cell to a population (e.g., Louvain-derived centers or the 14 known Baron populations)
- When the embedding method does not claim topology preservation (e.g., PCA/tSNE alone) — the short-distance relationships this analysis checks were demonstrated for scGAE, and competing methods projected ductal subtypes far apart

## Inputs

- preprocessed scRNA-seq expression matrix (scTransform-normalized, e.g., Seurat/scanpy output)
- cell population/cluster labels for each cell (e.g., the 14 Baron human pancreatic cell populations from GSE84133)
- low-dimensional embedding coordinates (20-dim scGAE latent space, embedding_coordinates.csv)

## Outputs

- per-population centroid coordinates in the 20-dimensional embedding
- all pairwise Euclidean centroid distances between the 14 populations (centroid_distances.csv)
- verification of the expected distance pattern (Fig. 4c,d): short activated-quiescent stellate and ductal-subtype distances relative to unrelated pairs

## How to apply

First embed the scRNA-seq data with scGAE: preprocess the count matrix with scTransform (in the Seurat package), build a cell KNN graph (default K=35, adjusted per dataset) from the preprocessed expression matrix, and train the multitask-oriented graph autoencoder (implemented with the Python package Spektral) to obtain a 20-dimensional latent embedding, saved as embedding_coordinates.csv. Then compute the centroid of each of the labeled cell populations in the embedding and calculate all pairwise Euclidean centroid distances, writing the result to centroid_distances.csv. Judge correctness by comparing the distance pattern to the paper's Fig. 4c,d expectations: the activated-quiescent stellate pair and the ductal-subtype pair should show shorter centroid distances than unrelated population pairs, and pairs projected far apart by competing methods like tSNE should remain short under scGAE. Note that scGAE's 2D visualization is produced by embedding to an intermediate dimension and applying tSNE, so centroid-distance analysis should be done in the intermediate (20-dim) latent space rather than the 2D tSNE plot.

## Related tools

- **scGAE** (graph autoencoder that embeds cells into the 20-dimensional latent space whose centroid distances are analyzed; implemented with the Python package Spektral) — github.com/ZixiangLuo1161/scGAE
- **Seurat (scTransform)** (preprocessing of the count matrix (e.g., Baron GSE84133) before KNN graph construction and embedding)
- **tSNE** (competitor/baseline method whose inter-population distances (e.g., ductal subtypes projected far apart) are compared against the scGAE centroid-distance pattern)

## Examples

```
adj = get_adj(data); model = SCGAE(); model.train(); model.clustering_train(centers); emb = model.embedding(); pd.DataFrame(emb).to_csv('embedding_coordinates.csv')
```

## Evaluation signals

- All pairwise centroid distances among the 14 populations are computed and written to centroid_distances.csv without missing entries
- Activated vs quiescent stellate cells have a centroid distance shorter than unrelated population pairs, matching Fig. 4c,d
- The two ductal subtypes remain at short centroid distance in the 20-dim embedding, unlike their far-apart projection under tSNE
- Embedding dimensionality equals 20 and the number of centroids equals the number of labeled populations (14 for the Baron dataset)
- The upstream KNN graph used default K=35 (adjusted per dataset) and preprocessing was done with scTransform in Seurat, as in the paper's protocol

## Limitations

- The optimal K for KNN graph construction varies among different datasets and different parts of a dataset, so graph quality (and hence the embedding and distances) depends on K choice
- No random seed or stochastic-variation control was reported, so centroid distances may vary across runs
- No software/package versions were specified for the tools (README lists versions, but the paper does not), complicating exact reproduction
- scGAE cannot directly embed into 2D; the centroid-distance analysis must be done in the intermediate dimension rather than a direct 2D graph-autoencoder embedding

## Evidence

- [readme] scGAE builds a cell graph and uses a multitask-oriented graph autoencoder to preserve topological structure information and feature information in scRNA-seq data simultaneously.: "scGAE builds a cell graph and uses a multitask-oriented graph autoencoder to preserve topological structure information and feature information in scRNA-seq data simultaneously."
- [discussion] The scRNA-seq data preprocessing was conducted using scTransform40 in The Seurat package41.: "The scRNA-seq data preprocessing was conducted using scTransform40 in The Seurat package41."
- [discussion] The default K is predened as 35 in this study and adjusted according to the datasets in our experiments.: "The default K is predened as 35 in this study and adjusted according to the datasets in our experiments."
- [discussion] we embedded the scRNA-seq data into an intermediate dimension and used tSNE to visualize the embedded data into a two-dimensional space.: "we embedded the scRNA-seq data into an intermediate dimension and used tSNE to visualize the embedded data into a two-dimensional space."
- [readme] Get the latent embedding of the data by model.embedding(): "Get the latent embedding of the data by model.embedding()"
