---
name: scrna-seq-latent-feature-learning
description: Use when scRNA-seq expression matrices (e.g., .npz of HVG features, AnnData)
  need cell-type-invariant latent representations for clustering, and deep generative
  baselines like scVI are too costly or hard to optimize.
when_to_use_negative:
- Datasets are small — Miscell demands a large sample size to obtain good performance,
  like other deep neural network methods.
- Data contain batch effects that have not been corrected — Miscell does not internally
  include any batch-correction methods; users must correct for batch effects before
  running it.
- The goal is marker gene discovery outside highly variable genes — Miscell's cluster-specific
  marker identification is limited to HVGs and may miss markers outside them.
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: Miscell
  role: Self-supervised contrastive learning (MoCo-style DenseNet encoder) that learns
    the latent cell feature representation
  repo: https://github.com/lixiangchun/Miscell
- name: FIt-SNE
  role: FFT-accelerated t-SNE 2D embedding of the learned latent features prior to
    DBSCAN clustering
  repo: https://github.com/KlugerLab/FIt-SNE
- name: Scanpy
  role: Neighborhood graph construction (n_neighbors=10, use_rep='X') and Leiden clustering
    of latent features; also used to normalize input features
- name: scikit-learn
  role: DBSCAN clustering (eps=1, min_samples=5) of the FIt-SNE embedding
- name: scVI
  role: Deep generative baseline method compared against Miscell's learned representations
- name: Seurat
  role: Baseline scRNA-seq clustering method compared against Miscell
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.isci.2021.103200
    title: 'Miscell: An efficient self-supervised learning approach for dissecting
      single-cell transcriptome'
  source_content_hash: sha256:cc728f111b669059f1eadbea833576ef8f6bfb92cd683c647c53e3a798449a5d
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/lixiangchun/Miscell
  repo_url: https://github.com/lixiangchun/Miscell
  tools:
  - FIt-SNE
  - Miscell
  - Scanpy
  - Seurat
  - scVI
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# scrna-seq-latent-feature-learning

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Learn latent cell representations from sparse, noisy single-cell RNA-seq data using Miscell, a self-supervised contrastive deep learning approach (DenseNet encoder, MoCo-style), and use the learned features for clustering and downstream embedding.

## When to use

Use when scRNA-seq expression matrices (e.g., .npz of HVG features, AnnData) need cell-type-invariant latent representations for clustering, and deep generative baselines like scVI are too costly or hard to optimize.

## When NOT to use

- Datasets are small — Miscell demands a large sample size to obtain good performance, like other deep neural network methods.
- Data contain batch effects that have not been corrected — Miscell does not internally include any batch-correction methods; users must correct for batch effects before running it.
- The goal is marker gene discovery outside highly variable genes — Miscell's cluster-specific marker identification is limited to HVGs and may miss markers outside them.

## Inputs

- Normalized single-cell gene expression feature matrix (npz, e.g. data/x_new_scanpy_HVG.npz with 'x' array of HVG features)
- AnnData object with ground-truth cell-type labels (e.g., 55,656-cell Tabula Muris Senis from Figshare project 64982)
- Pretrained Miscell checkpoint (checkpoint_*.pth.tar)

## Outputs

- Latent feature matrix per cell (moco-dim=64 dimensional embeddings)
- Cluster assignments from DBSCAN on FIt-SNE embedding or Scanpy Leiden labels
- t-SNE visualization colored by Leiden clusters (figures/tsne_scanpy.pdf)
- Clustering metric comparison table (clustering_metrics.csv: ARI, NMI, V-measure)

## How to apply

Normalize the feature matrix (e.g., Scanpy-normalized HVG matrix such as x_new_scanpy_HVG.npz with in_features matching the gene count, e.g. 1686) and train a Miscell DenseNet encoder (e.g. densenet11) with MoCo contrastive learning (moco-t 0.2, moco-k 1024, moco-m 0.999, moco-dim 64, batch 256, lr 0.24) on GPUs; at least two GPUs are recommended. Extract features from a pretrained checkpoint (e.g. checkpoint_0199.pth.tar) using utils.load_pretrained_model and utils.extract_features. Embed the learned features with FIt-SNE (seed=123, nthreads=12, perplexity_list=[30, 36, 42, 48]) and cluster with DBSCAN (eps=1, min_samples=5) or with Scanpy (sc.pp.neighbors n_neighbors=10 on 'X', then Leiden). Compare clustering quality (ARI, NMI, V-measure) against Seurat, Scanpy, and scVI baselines against ground-truth cell-type labels. Batch correction is NOT internal — correct batch effects as a preprocessing step before training.

## Related tools

- **Miscell** (Self-supervised contrastive learning (MoCo-style DenseNet encoder) that learns the latent cell feature representation) — https://github.com/lixiangchun/Miscell
- **FIt-SNE** (FFT-accelerated t-SNE 2D embedding of the learned latent features prior to DBSCAN clustering) — https://github.com/KlugerLab/FIt-SNE
- **Scanpy** (Neighborhood graph construction (n_neighbors=10, use_rep='X') and Leiden clustering of latent features; also used to normalize input features)
- **scikit-learn** (DBSCAN clustering (eps=1, min_samples=5) of the FIt-SNE embedding)
- **scVI** (Deep generative baseline method compared against Miscell's learned representations)
- **Seurat** (Baseline scRNA-seq clustering method compared against Miscell)

## Examples

```
python main.py -a densenet11 -j 1 -b 256 --lr 0.24 --dist-url "tcp://localhost:10001" --world-size 1 --rank 0 --multiprocessing-distributed --outdir checkpoint --in_features 1686 --mlp --moco-t 0.2 --moco-k 1024 --moco-m 0.999 --moco-dim 64 data/x_new_scanpy_HVG.npz
```

## Evaluation signals

- Miscell's clustering must achieve at least a +10% ARI gain over the second-best method with comparable NMI and V-measure (e.g., on the 55,656-cell Tabula Muris dataset).
- in_features passed to the model must equal the number of columns of the input npz 'x' array (assert in_features == X.shape[1]).
- The FIt-SNE embedding is reproducible: fast_tsne must be called with seed=123, nthreads=12, perplexity_list=[30, 36, 42, 48].
- Leiden cluster assignments must be present in adata.obs['leiden'] after sc.pp.neighbors(n_neighbors=10, use_rep='X') and sc.tl.leiden, and the t-SNE plot saved as figures/tsne_scanpy.pdf.
- clustering_metrics.csv must contain ARI, NMI, and V-measure rows for Miscell, Scanpy, Seurat, and scVI computed against the same reference labels.

## Limitations

- Miscell demands a large sample size to obtain good performance, limiting use on small datasets.
- Miscell does not internally include any batch-correction methods; users must correct for batch effects before running it.
- Clinically significant latent features extracted by Miscell lack explainability.
- Cluster-specific marker gene identification is limited to highly variable genes, so markers outside HVGs may be missed.
- Training requires GPUs — at least two GPUs are required for the example training command.

## Evidence
