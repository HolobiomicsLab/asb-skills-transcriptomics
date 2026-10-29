---
name: scrna-seq-self-supervised-representation-learning
description: Use when you need to cluster or embed a large scRNA-seq dataset (e.g.
  tens of thousands of cells, like the 43,095-cell 10x PBMC dataset) and standard
  pipelines underperform, or to benchmark deep-learning representation learning against
  Scanpy/Seurat/scVI.
when_to_use_negative:
- Datasets are small — Miscell demands a large sample size to obtain good performance
- Data contain batch effects you cannot correct beforehand — Miscell does not internally
  include any batch-correction methods
- You need cluster-specific marker genes outside the highly variable gene set — Miscell's
  marker identification is limited to HVGs
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3474
tools:
- name: Miscell
  role: Self-supervised contrastive model (MoCo-based) with deep neural network latent
    feature encoder trained on single-cell transcriptomes
  repo: https://github.com/lixiangchun/Miscell
- name: FIt-SNE
  role: FFT-accelerated interpolation-based t-SNE used to embed Miscell latent features
    before DBSCAN clustering
  repo: https://github.com/KlugerLab/FIt-SNE
- name: Scanpy
  role: Used for data normalization (HVG matrix), building the neighborhood graph
    (n_neighbors=10) and Leiden clustering; also a baseline pipeline
- name: Seurat
  role: State-of-the-art baseline clustering method compared against Miscell via ARI/NMI/V-measure
- name: scVI
  role: Deep generative baseline; train scVI and cluster cells in its learned latent
    space for comparison
- name: scikit-learn
  role: DBSCAN clustering of the t-SNE embedding (eps=1, min_samples=5) and metric
    computation
- name: facebookresearch/moco
  role: Source of the MoCo contrastive-learning code copied into Miscell (--moco-t,
    --moco-k, --moco-m, --moco-dim flags)
  repo: https://github.com/facebookresearch/moco
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.isci.2021.103200
    title: 'Miscell: An efficient self-supervised learning approach for dissecting
      single-cell transcriptome'
  source_content_hash: sha256:f858821ea60ee3663370e6ba47009e8cd31c3af6a7beb61865c9a1d3a23f5cf7
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
  - facebookresearch/moco
  - scVI
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 14
---

# scrna-seq-self-supervised-representation-learning

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Train Miscell, a MoCo-style self-supervised contrastive model with a deep neural network (DenseNet) latent feature encoder, on sparse single-cell RNA-seq count matrices to learn cell-type invariant representations that outperform Seurat, Scanpy and scVI on clustering (ARI, NMI, V-measure).

## When to use

Use when you need to cluster or embed a large scRNA-seq dataset (e.g. tens of thousands of cells, like the 43,095-cell 10x PBMC dataset) and standard pipelines underperform, or to benchmark deep-learning representation learning against Scanpy/Seurat/scVI.

## When NOT to use

- Datasets are small — Miscell demands a large sample size to obtain good performance
- Data contain batch effects you cannot correct beforehand — Miscell does not internally include any batch-correction methods
- You need cluster-specific marker genes outside the highly variable gene set — Miscell's marker identification is limited to HVGs

## Inputs

- Normalized scRNA-seq count matrix restricted to highly variable genes, stored as .npz (e.g. data/x_new_scanpy_HVG.npz with key 'x')
- Number of input features (in_features, e.g. 1686) matching the matrix column count
- Reference cell-type labels for metric computation (ARI, NMI, V-measure)

## Outputs

- Trained Miscell checkpoint (e.g. checkpoint_0199.pth.tar)
- Latent feature matrix for every cell extracted from the pretrained model
- FIt-SNE 2D embedding (seed=123, nthreads=12, perplexity_list=[30,36,42,48])
- Cluster assignments from DBSCAN (eps=1, min_samples=5) or Leiden clustering
- t-SNE plot colored by Leiden clusters (t-SNE_scanpy.pdf)
- Per-method clustering metrics table (clustering_metrics.csv: ARI, NMI, V-measure for Miscell vs Scanpy, Seurat, scVI)

## How to apply

Normalize the single-cell expression matrix (e.g. Scanpy-normalized highly variable gene matrix such as data/x_new_scanpy_HVG.npz) and restrict it to a fixed number of input features (in_features, e.g. 1686). Train the Miscell model on GPUs (at least two required) with a densenet arch (densenet11/21/29/63), MoCo hyperparameters (--moco-t 0.2 --moco-k 1024 --moco-m 0.999 --moco-dim 64, batch 256, lr 0.24) and contrastive learning between original and augmented expression profiles. Extract latent features per cell with the pretrained checkpoint (e.g. checkpoint_0199.pth.tar), then embed them with FIt-SNE (seed=123, nthreads=12, perplexity_list=[30,36,42,48]) and cluster the embedding with DBSCAN (eps=1, min_samples=5) or a Scanpy neighbor graph (n_neighbors=10, use_rep='X') followed by Leiden clustering. Compare against Scanpy's preprocessing-and-Leiden pipeline, Seurat, and scVI using adjusted rand index, normalized mutual information, and V-measure against reference cell-type labels — Miscell reported gains of 33% ARI, 6% NMI, and 7% V-measure over the second-best method on PBMC. Batch correction is NOT done by Miscell; correct batch effects as a preprocessing step beforehand.

## Related tools

- **Miscell** (Self-supervised contrastive model (MoCo-based) with deep neural network latent feature encoder trained on single-cell transcriptomes) — https://github.com/lixiangchun/Miscell
- **FIt-SNE** (FFT-accelerated interpolation-based t-SNE used to embed Miscell latent features before DBSCAN clustering) — https://github.com/KlugerLab/FIt-SNE
- **Scanpy** (Used for data normalization (HVG matrix), building the neighborhood graph (n_neighbors=10) and Leiden clustering; also a baseline pipeline)
- **Seurat** (State-of-the-art baseline clustering method compared against Miscell via ARI/NMI/V-measure)
- **scVI** (Deep generative baseline; train scVI and cluster cells in its learned latent space for comparison)
- **scikit-learn** (DBSCAN clustering of the t-SNE embedding (eps=1, min_samples=5) and metric computation)
- **facebookresearch/moco** (Source of the MoCo contrastive-learning code copied into Miscell (--moco-t, --moco-k, --moco-m, --moco-dim flags)) — https://github.com/facebookresearch/moco

## Examples

```
python main.py -a densenet11 -j 1 -b 256 --lr 0.24 --dist-url "tcp://localhost:10001" --world-size 1 --rank 0 --multiprocessing-distributed --outdir checkpoint --in_features 1686 --save_frequency 5 --mlp --moco-t 0.2 --moco-k 1024 --moco-m 0.999 --moco-dim 64 data/x_new_scanpy_HVG.npz
```

## Evaluation signals

- Miscell's clustering assignments must yield higher adjusted rand index, normalized mutual information, and V-measure than Scanpy, Seurat and scVI (reported gains: ARI +33%, NMI +6%, V-measure +7% on PBMC)
- in_features passed to the model must equal X.shape[1] of the loaded .npz feature matrix (assert in_features == X.shape[1])
- FIt-SNE embedding must use seed=123, nthreads=12 and perplexity_list=[30,36,42,48]; DBSCAN must use eps=1, min_samples=5
- t-SNE plot colored by Leiden clusters is saved as t-SNE_scanpy.pdf
- Training requires at least two GPUs and produces a checkpoint (e.g. checkpoint_0199.pth.tar); features extracted with return_feature=True

## Limitations

- Miscell demands a large sample size to obtain good performance, limiting use on small datasets
- Miscell does not internally include any batch-correction methods; users must correct batch effects before running it
- Clinically significant latent features lack explainability; future work required
- Cluster-specific marker gene identification is limited to highly variable genes, so markers outside HVGs may be missed
- Deep generative alternatives like scVI/MARS rely on reconstruction loss that is difficult to optimize and computationally costly — a motivation for Miscell but relevant when choosing baselines

## Evidence
