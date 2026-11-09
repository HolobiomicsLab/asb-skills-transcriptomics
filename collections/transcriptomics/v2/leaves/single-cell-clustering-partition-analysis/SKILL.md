---
name: single-cell-clustering-partition-analysis
description: Use when raw multimodal single-cell count data (RNA matrix X1 plus ADT
  matrix X2, e.g. h5 from GSE128639/bmcite) must be partitioned into unlabeled cell
  clusters and characterized by per-cluster marker/gene-set analysis.
when_to_use_negative:
- Input is already a normalized embedding, cluster-label table, or feature matrix
  — the skill expects raw count matrices (X1 RNA / X2 ADT) in h5 that are normalized
  with Scanpy
- The analysis goal is joint embedding or label transfer with a reference rather than
  unsupervised clustering — the article notes joint-embedding-first methods are not
  designed and optimized for clustering, and label transfer is a separate workflow
- You need automatic estimation of the cluster number — K must be supplied by the
  user after pretraining
edam_operation: http://edamontology.org/operation_2930
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3391
tools:
- name: scMDC
  role: End-to-end multimodal deep clustering model that jointly learns latent features
    for partitioning single-cell multi-omics data
  repo: https://github.com/xianglin226/scMDC
- name: Scanpy
  role: Preprocessing and separate normalization of mRNA and ADT count data (filtering,
    size-factor normalization, log transform, scaling)
- name: Pytorch
  role: Deep learning framework for training the ZINB autoencoder and deep K-means
    clustering stage
- name: h5py
  role: Loading the preprocessed h5 input containing RNA counts X1 and ADT counts
    X2
- name: fgsea
  role: Runs gene set enrichment analysis on the directed gene rank per cluster (v1.19.4)
- name: msigdbr
  role: Supplies Hallmark gene sets for the fgsea analysis (v7.4.1)
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-022-35031-9
    title: Clustering of single-cell multi-omics data with a multimodal deep learning
      method
  source_content_hash: sha256:85f83ffb430b132cd5a264e755d03a7094077d498b5b17d34f8d745fe3a9aec2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xianglin226/scMDC
  tools:
  - Pytorch
  - Scanpy
  - fgsea
  - h5py
  - msigdbr
  - scMDC
  edam_operation: http://edamontology.org/operation_2930
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3391
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# single-cell-clustering-partition-analysis

## Summary

Unsupervised partitioning of single-cell multi-omics counts (scRNA-seq plus ADT from CITE-seq) into cell clusters using scMDC, a multimodal deep learning model with ZINB reconstruction losses, followed by deep differential analysis per cluster.

## When to use

Use when raw multimodal single-cell count data (RNA matrix X1 plus ADT matrix X2, e.g. h5 from GSE128639/bmcite) must be partitioned into unlabeled cell clusters and characterized by per-cluster marker/gene-set analysis.

## When NOT to use

- Input is already a normalized embedding, cluster-label table, or feature matrix — the skill expects raw count matrices (X1 RNA / X2 ADT) in h5 that are normalized with Scanpy
- The analysis goal is joint embedding or label transfer with a reference rather than unsupervised clustering — the article notes joint-embedding-first methods are not designed and optimized for clustering, and label transfer is a separate workflow
- You need automatic estimation of the cluster number — K must be supplied by the user after pretraining

## Inputs

- h5 file with preprocessed RNA counts X1 and ADT counts X2 (Scanpy-normalized), e.g. GSE128639 BMNC/bmcite
- Pretrained scMDC autoencoder weights (--ae_weights)
- User-specified number of clusters K
- Hallmark gene sets from msigdbr (v7.4.1) for downstream GSEA

## Outputs

- scMDC cluster assignments per cell for the four largest BMNC clusters (e.g. CD14 monocytes)
- Per-cluster directed gene ranks from LRP minimal-perturbation differential analysis
- fgsea_hallmark_gsea_results.csv with per-cluster NES and adjusted p-values
- Dot plots of enriched Hallmark pathways per cluster and a pathway heatmap (Fig. 8 style)

## How to apply

Prepare the input in h5 format containing RNA counts X1 and ADT counts X2 (e.g. GSE128639 BMNC loaded with h5py), preprocessing and normalizing mRNA and ADT separately with Scanpy (filter zero-count genes/ADTs, size-factor normalization, log transform, scaling). Train scMDC: a denoising multimodal autoencoder with zero-inflated negative binomial (ZINB) reconstruction loss is pretrained for 400 epochs (reconstruction first 200 epochs, KL loss added for the remaining 200), then K centroids are initialized by K-means on the pretrained latent space and autoencoder, KL-loss, and deep K-means are optimized simultaneously in the clustering stage. The user must specify the number of clusters K since scMDC provides no automatic estimation. After clustering, run the LRP deep differential analysis (run_LRP.py) per cluster against the rest to obtain a directed gene rank from minimal-perturbation gene importances, then optionally feed that rank to fgsea with msigdbr Hallmark gene sets in R. For BMNC the four largest clusters (including CD14 monocytes) are analyzed downstream.

## Related tools

- **scMDC** (End-to-end multimodal deep clustering model that jointly learns latent features for partitioning single-cell multi-omics data) — https://github.com/xianglin226/scMDC
- **Scanpy** (Preprocessing and separate normalization of mRNA and ADT count data (filtering, size-factor normalization, log transform, scaling))
- **Pytorch** (Deep learning framework for training the ZINB autoencoder and deep K-means clustering stage)
- **h5py** (Loading the preprocessed h5 input containing RNA counts X1 and ADT counts X2)
- **fgsea** (Runs gene set enrichment analysis on the directed gene rank per cluster (v1.19.4))
- **msigdbr** (Supplies Hallmark gene sets for the fgsea analysis (v7.4.1))

## Examples

```
import h5py; f = h5py.File('bmnc.h5', 'r'); X1 = f['X1'][:]; X2 = f['X2'][:]  # load Scanpy-preprocessed RNA and ADT counts before scMDC clustering
```

## Evaluation signals

- fgsea_hallmark_gsea_results.csv exists and contains per-cluster NES and padj fields
- Dot plots and heatmap of enriched Hallmark pathways render consistently with the published Fig. 8
- The complement system pathway has the highest enrichment score in CD14 monocytes for the BMNC Hallmark GSEA
- Cluster assignments on BMNC partition cells into biologically coherent groups matching the four largest clusters used in downstream DE analysis
- Running time scales linearly with dataset size, as reported for scMDC on large multimodal datasets

## Limitations

- The number of clusters K must be specified by the user; scMDC provides no automatic estimation
- All experiments require GPU hardware (Nvidia Tesla P100, 16 G), which may limit reproducibility on CPU-only systems
- No explicit limitations or caveats subsection is present in the provided article text

## Evidence

- [abstract] scMDC develops multimodal clustering: "we develop a novel multimodal deep learning method, scMDC, for single-cell multi-omics data clustering analysis"
- [discussion] Preprocessing with Scanpy: "The raw CITE-seq data is preprocessed and normalized by the Python package SCANPY48. mRNA and ADT data are normalized separately but using the same method."
- [discussion] Autoencoder pretraining epochs: "We pretrain the autoencoders for 400 epochs before entering the clustering stage."
- [discussion] K-means centroid initialization: "we initialize K centroids by implementing K-means algorithm on the pretrained latent space."
- [discussion] User must specify K: "After pretraining, the users need to specify the number of clusters (K)."
- [methods] h5 input preparation: "1) Prepare the input data in h5 format."
