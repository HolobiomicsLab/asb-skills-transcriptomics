---
name: scrna-seq-cluster-marker-validation
description: Use when a pretrained self-supervised scRNA-seq encoder (Miscell) has
  produced cell clusters and you need ranked, cluster-specific marker genes via attribution
  rather than differential-expression tests.
when_to_use_negative:
- The marker genes of interest may lie outside the highly variable gene set, since
  Miscell's attribution-based identification is limited to HVGs
- The dataset is small, because Miscell (like deep neural network methods generally)
  demands a large sample size to obtain good performance
- The input data have batch effects that have not been corrected, since Miscell does
  not internally include any batch-correction methods
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: Miscell
  role: Self-supervised deep neural network encoder whose frozen weights feed the
    linear classifier for integrated-gradient marker attribution; installed from the
    GitHub repo and trained via main.py with --mlp --moco-t 0.2 --moco-k 1024 --moco-m
    0.999 --moco-dim 64
  repo: github.com/lixiangchun/Miscell
- name: Scanpy
  role: Used for the comparison differential analysis (t test, Wilcoxon rank-sum test)
    and for Leiden clustering of the extracted latent features prior to marker attribution
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.isci.2021.103200
    title: 'Miscell: An efficient self-supervised learning approach for dissecting
      single-cell transcriptome'
  source_content_hash: sha256:c125879ac9aa4ef93beb6ce18adf8f1f2513f84426da9d44a8250ad9dbb34a4b
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/lixiangchun/Miscell
  tools:
  - Miscell
  - Scanpy
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 15
---

# scrna-seq-cluster-marker-validation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Identify cluster-specific marker genes from a trained Miscell self-supervised encoder by freezing its weights, attaching a 1-layer linear classifier, and applying integrated-gradients attribution to rank each gene's influence per cell cluster.

## When to use

Use when a pretrained self-supervised scRNA-seq encoder (Miscell) has produced cell clusters and you need ranked, cluster-specific marker genes via attribution rather than differential-expression tests.

## When NOT to use

- The marker genes of interest may lie outside the highly variable gene set, since Miscell's attribution-based identification is limited to HVGs
- The dataset is small, because Miscell (like deep neural network methods generally) demands a large sample size to obtain good performance
- The input data have batch effects that have not been corrected, since Miscell does not internally include any batch-correction methods

## Inputs

- Pretrained Miscell encoder checkpoint (e.g., checkpoint_0199.pth.tar from the --outdir checkpoint directory)
- Normalized highly-variable-gene expression matrix in .npz format (e.g., data/x_new_scanpy_HVG.npz with in_features=1686 for the Smart-seq HVG set)
- Cell cluster labels (from sklearn.cluster.dbscan on FIt-SNE features or scanpy Leiden clustering)
- Gene name list corresponding to the HVG input feature columns

## Outputs

- Gene-by-cluster integrated-gradient attribution score table written to integrated_gradient_scores.csv
- Ranked list of top cluster-specific marker genes per cluster

## How to apply

Load the pretrained Miscell encoder (e.g., a densenet11 checkpoint such as checkpoint_0199.pth.tar) and freeze its weights so the self-supervised latent representation is preserved. Attach a single-layer linear classifier on top of the frozen encoder and train only that classifier on the cell cluster labels obtained from DBSCAN-on-FIt-SNE or Scanpy Leiden clustering. Compute integrated-gradient attribution scores quantifying each gene's influence on each cluster's classification through the frozen encoder + linear classifier, then rank genes by attribution score within each cluster and extract the top-ranked cluster-specific markers. Note that marker discovery is restricted to the highly variable genes (HVGs) supplied in the input matrix (e.g., data/x_new_scanpy_HVG.npz), and batch correction must be done as a separate preprocessing step before encoding.

## Related tools

- **Miscell** (Self-supervised deep neural network encoder whose frozen weights feed the linear classifier for integrated-gradient marker attribution; installed from the GitHub repo and trained via main.py with --mlp --moco-t 0.2 --moco-k 1024 --moco-m 0.999 --moco-dim 64) — github.com/lixiangchun/Miscell
- **Scanpy** (Used for the comparison differential analysis (t test, Wilcoxon rank-sum test) and for Leiden clustering of the extracted latent features prior to marker attribution)

## Examples

```
import utils, numpy as np; X = np.load('data/x_new_scanpy_HVG.npz')['x']; model = utils.load_pretrained_model('densenet11', 1686, 'checkpoint_0199.pth.tar', return_feature=True); features = utils.extract_features(model, X)
```

## Evaluation signals

- On the Smart-seq dataset the recovered CD4+ T cell markers include CD4, CD40LG, and CCR4, matching the calls made by Scanpy and Seurat
- On the Smart-seq dataset the recovered CD8+ T cell markers include CD8A, CD8B, and KLRK1 (plus XCL1), matching Scanpy and Seurat
- Cluster-specific calls reproduce the reported markers: SLAMF7/PRF1 for cluster C29, TIGIT/CCDC22 for C49, XCR1/SLC4A10/CTLA4 for C8, and KLRG1 for C53
- The full gene-by-cluster attribution table exists as integrated_gradient_scores.csv with one attribution score per gene per cluster
- All genes in the attribution output fall within the input HVG set (e.g., the 1686 in_features columns of x_new_scanpy_HVG.npz)

## Limitations

- Cluster-specific marker gene identification by Miscell is limited to the highly variable genes, so markers outside HVGs may be missed
- Miscell demands a large sample size to obtain good performance, limiting use on small datasets
- Miscell does not internally include any batch-correction methods; users must correct batch effects before running it
- The clinically significant latent features extracted by Miscell lack explainability and future work is required

## Evidence
