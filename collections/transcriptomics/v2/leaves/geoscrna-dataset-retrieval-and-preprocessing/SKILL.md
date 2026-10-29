---
name: geoscrna-dataset-retrieval-and-preprocessing
description: Use when starting a scRNA-seq analysis from a public GEO accession (raw
  UMI count matrix) and a graph-based deep-learning method like scGAE requires scTransform-preprocessed
  input data.
when_to_use_negative:
- The input is already an scTransform-normalized/preprocessed matrix or an embedding
  — re-running retrieval and preprocessing would discard prior work.
- The analysis target is not scRNA-seq count data (e.g., bulk RNA-seq, spatial or
  proteomics data) — scGAE's cell-graph preprocessing assumes single-cell count matrices.
- You need direct 2D embedding output from the autoencoder — the paper states the
  complex data structure is hard to embed directly into 2D and instead embeds to an
  intermediate dimension followed by tSNE.
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: scGAE
  role: Downstream consumer of the preprocessed data; its README defines the preprocessing
    contract (get_adj → SCGAE model → train → embedding) and version-pinned dependencies
  repo: github.com/ZixiangLuo1161/scGAE
- name: t-SNE
  role: Used after scGAE embeds data into an intermediate dimension to visualize the
    embedded data in two-dimensional space
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41598-021-99003-7
    title: A topology-preserving dimensionality reduction method for single-cell RNA-seq
      data using graph autoencoder
  source_content_hash: sha256:3cc8d7c9462a4d70d2392c12b3163265d01dd4b893caa386b016603b0524ec27
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - scGAE
  - t-SNE
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# geoscrna-dataset-retrieval-and-preprocessing

## Summary

Retrieves a public scRNA-seq count matrix from GEO (e.g., GSE84133 Baron human pancreas) and preprocesses it with scTransform in the Seurat package so the normalized matrix is in the exact form scGAE expects before KNN graph construction and graph-autoencoder embedding.

## When to use

Use when starting a scRNA-seq analysis from a public GEO accession (raw UMI count matrix) and a graph-based deep-learning method like scGAE requires scTransform-preprocessed input data.

## When NOT to use

- The input is already an scTransform-normalized/preprocessed matrix or an embedding — re-running retrieval and preprocessing would discard prior work.
- The analysis target is not scRNA-seq count data (e.g., bulk RNA-seq, spatial or proteomics data) — scGAE's cell-graph preprocessing assumes single-cell count matrices.
- You need direct 2D embedding output from the autoencoder — the paper states the complex data structure is hard to embed directly into 2D and instead embeds to an intermediate dimension followed by tSNE.

## Inputs

- GEO accession count matrix (e.g., Baron human pancreas GSE84133, raw UMI counts)
- Cell-by-gene count matrix (CSV/h5ad/AnnData) suitable for scTransform normalization
- Choice of KNN neighbor count (default K=35, adjusted per dataset)

## Outputs

- scTransform-preprocessed expression matrix in a form accepted by scGAE (Seurat/scanpy object)
- Cell adjacency matrix produced by get_adj for KNN graph construction (default K=35)
- Preprocessed dataset ready for SCGAE model building, training, and embedding generation

## How to apply

Download the count matrix from the GEO accession named in the paper (pancreatic cells data under GSE84133). Preprocess the count matrix with scTransform in the Seurat R package, following the paper's preprocessing protocol; the scGAE README notes preprocessing can equivalently be done with the Seurat R package or the scanpy Python package, and that scGAE takes preprocessed data as input. From the preprocessed expression matrix, compute the adjacency matrix using the get_adj function to build the cell graph with a KNN algorithm (default K=35 in the study, adjusted per dataset). Only then build and train the SCGAE model (TensorFlow 2.3.0, Spektral 0.6.1 per the README requirements) and obtain the latent embedding via model.embedding(). Save intermediate artifacts (e.g., embedding_coordinates.csv) so downstream centroid-distance checks can be re-run.

## Related tools

- **scGAE** (Downstream consumer of the preprocessed data; its README defines the preprocessing contract (get_adj → SCGAE model → train → embedding) and version-pinned dependencies) — github.com/ZixiangLuo1161/scGAE
- **t-SNE** (Used after scGAE embeds data into an intermediate dimension to visualize the embedded data in two-dimensional space)

## Examples

```
import scgae; adj = scgae.get_adj(data); model = scgae.SCGAE(adj, data); model.train(); emb = model.embedding()
```

## Evaluation signals

- The downloaded GEO matrix matches the accession named in the paper (pancreatic cells data, GSE84133) and contains raw counts amenable to scTransform.
- The KNN graph is built from the preprocessed (not raw) matrix, with the adjacency matrix produced by get_adj and default K=35.
- The environment matches the README requirements (tensorflow 2.3.0, spektral 0.6.1, scanpy 1.8.0, anndata 0.7.4) so SCGAE model construction and model.train() run without version errors.
- The pipeline produces the documented artifacts (e.g., embedding_coordinates.csv for the 20-dimensional embedding) so downstream centroid-distance verification is possible.
- Downstream check: the embedding preserves topological structure, e.g., activated-quiescent stellate and ductal-subtype population pairs show shorter centroid distances than unrelated pairs (Fig. 4c,d pattern).

## Limitations

- The optimal K for KNN graph construction varies among different datasets and different parts of a dataset; the default K=35 must be adjusted per dataset.
- The article specifies no software/package versions, random seed, or stochastic-variation control, so reproducibility relies on the README's version pins (tensorflow 2.3.0, spektral 0.6.1, etc.).
- scGAE does not embed directly into 2D — data go to an intermediate dimension and then tSNE, and the authors note a better 2D visualization method is still needed.

## Evidence

- [readme] scGAE takes preprocessed data as input. Single cell data preprocessing can be done with Seurat R package and scanpy python package.: "scGAE takes preprocessed data as input. Single cell data preprocessing can be done with Seurat R package and scanpy python package."
- [discussion] The scRNA-seq data preprocessing was conducted using scTransform in The Seurat package.: "The scRNA-seq data preprocessing was conducted using scTransform40 in The Seurat package41."
- [discussion] The default K is predened as 35 in this study and adjusted according to the datasets in our experiments.: "The default K is predened as 35 in this study and adjusted according to the datasets in our experiments."
- [readme] Compute the adjacency matrix using get_adj function and build the SCGAE model.: "Compute the adjacency matrix using get_adj function and build the SCGAE model."
