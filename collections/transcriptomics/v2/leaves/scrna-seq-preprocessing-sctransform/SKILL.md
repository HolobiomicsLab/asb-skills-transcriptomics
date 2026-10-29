---
name: scrna-seq-preprocessing-sctransform
description: Use when the input is a raw single-cell RNA-seq count matrix (e.g. from
  NGDC HRA000084 or GEO GSE84133) that must be preprocessed before building a cell-to-cell
  KNN graph and training a graph autoencoder.
when_to_use_negative:
- The expression matrix has already been normalized/preprocessed — re-running scTransform
  would double-correct the data before scGAE model building.
- The data are bulk RNA-seq or other non-single-cell expression profiles, for which
  scTransform's single-cell noise model does not apply.
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: scGAE
  role: 'Downstream consumer of the scTransform-preprocessed matrix: builds the cell
    graph (get_adj, KNN K=35) and trains the multitask-oriented graph autoencoder'
  repo: https://github.com/ZixiangLuo1161/scGAE
- name: Seurat
  role: R package whose scTransform implementation performs the count-matrix normalization
    step
- name: scanpy
  role: Alternative Python package (v1.8.0 in the reference environment) for single-cell
    preprocessing before scGAE
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41598-021-99003-7
    title: A topology-preserving dimensionality reduction method for single-cell RNA-seq
      data using graph autoencoder
  source_content_hash: sha256:9493ba3c2584a6b21bec98ef53ef36bda52d88831594f6c8cf85682877b3d8c2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ZixiangLuo1161/scGAE
  tools:
  - Seurat
  - scGAE
  - scanpy
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# scrna-seq-preprocessing-sctransform

## Summary

Normalize and variance-stabilize a raw scRNA-seq count matrix with scTransform (from the Seurat R package) so downstream graph-construction methods like scGAE receive corrected, model-ready expression features.

## When to use

Use when the input is a raw single-cell RNA-seq count matrix (e.g. from NGDC HRA000084 or GEO GSE84133) that must be preprocessed before building a cell-to-cell KNN graph and training a graph autoencoder.

## When NOT to use

- The expression matrix has already been normalized/preprocessed — re-running scTransform would double-correct the data before scGAE model building.
- The data are bulk RNA-seq or other non-single-cell expression profiles, for which scTransform's single-cell noise model does not apply.

## Inputs

- Raw scRNA-seq gene-by-cell count matrix (e.g. HSPC data, NGDC accession HRA000084)
- Cell/gene metadata annotations (e.g. prior Ba/Eo/MaP labels for downstream re-clustering)

## Outputs

- scTransform-normalized expression matrix suitable as input to scGAE's get_adj / SCGAE model construction
- Preprocessed object compatible with scanpy/anndata (anndata 0.7.4) workflows

## How to apply

Take the raw count matrix downloaded for the dataset of interest (e.g. the HSPC scRNA-seq data under accession HRA000084 from the National Genomics Data Center) and run scTransform in the Seurat R package, exactly as done in the scGAE study, to produce a normalized, variance-stabilized expression matrix. Alternatively, preprocessing can be done with the scanpy Python package, since the scGAE README notes preprocessing 'can be done with Seurat R package and scanpy python package'. Feed the preprocessed matrix into the downstream skill's workflow: compute the cell-to-cell adjacency with the get_adj function (KNN, default K=35), train the multitask-oriented graph autoencoder, and extract the latent embedding. The rationale is that massively parallel scRNA-seq data are high-dropout and high-noise, so uncorrected counts would corrupt the topological structure the graph autoencoder is meant to preserve.

## Related tools

- **scGAE** (Downstream consumer of the scTransform-preprocessed matrix: builds the cell graph (get_adj, KNN K=35) and trains the multitask-oriented graph autoencoder) — https://github.com/ZixiangLuo1161/scGAE
- **Seurat** (R package whose scTransform implementation performs the count-matrix normalization step)
- **scanpy** (Alternative Python package (v1.8.0 in the reference environment) for single-cell preprocessing before scGAE)

## Examples

```
# R / Seurat: normalize HSPC counts (HRA000084) with scTransform before scGAE
library(Seurat); seu <- CreateSeuratObject(counts = counts_matrix); seu <- SCTransform(seu)
```

## Evaluation signals

- The preprocessed matrix, not raw counts, is what feeds get_adj / SCGAE model construction in the Run_scGAE.ipynb workflow.
- Environment matches the README requirements (e.g. scanpy 1.8.0, anndata 0.7.4) so preprocessing output round-trips into the scGAE pipeline.
- Downstream sanity check: after scGAE embedding + tSNE, previously annotated Ba/Eo/MaP cells should resolve into at least 2 Louvain subclusters (Fig. 4a,b structure).
- Preprocessing should reduce dropout/noise-driven dispersion so that the KNN cell graph (default K=35) yields a connected, cluster-preserving adjacency.

## Limitations

- No software/package versions are specified in the paper itself for the preprocessing step (the README lists versions for the Python environment only).
- The optimal K for the subsequent KNN graph varies among datasets and dataset regions, so preprocessing choices cannot be fixed independently of the downstream graph step.
- A better 2D visualization than tSNE is still needed, so preprocessing quality cannot be fully judged from the 2D tSNE plot alone; scGAE cannot directly embed into 2D.

## Evidence

- [discussion] Preprocess count matrix using scTransform: "The scRNA-seq data preprocessing was conducted using scTransform40 in The Seurat package41."
- [readme] Preprocessing tools per README: "Single cell data preprocessing can be done with Seurat R package and scanpy python package."
- [intro] Data-quality motivation: "the data generated by massively parallel scRNA-seq are of high dropout and high noise with complex structure, which posed a series of challenges on dimensionality reduction"
- [discussion] Downstream graph construction: "The default K is predened as 35 in this study and adjusted according to the datasets in our experiments."
