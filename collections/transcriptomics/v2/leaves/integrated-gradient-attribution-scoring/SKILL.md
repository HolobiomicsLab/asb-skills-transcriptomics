---
name: integrated-gradient-attribution-scoring
description: Use when cluster-specific marker genes must be recovered from a trained
  self-supervised scRNA-seq encoder (e.g., Miscell latent features) and attributed
  to individual genes via an attribution method.
when_to_use_negative:
- The gene of interest is not among the highly variable genes, since Miscell's marker
  identification is limited to HVGs and may miss cluster-specific markers outside
  them
- The dataset is too small, because Miscell demands a large sample size to obtain
  good performance
- Batch effects have not been corrected beforehand, since Miscell does not internally
  include any batch-correction method
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3474
tools:
- name: Miscell
  role: Self-supervised encoder whose frozen latent representation feeds the linear
    classifier used for integrated-gradient marker attribution
  repo: https://github.com/lixiangchun/Miscell
- name: Scanpy
  role: Clustering/neighborhood-graph framework whose Leiden cluster labels train
    the linear classifier and serve as reference marker calls
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.isci.2021.103200
    title: 'Miscell: An efficient self-supervised learning approach for dissecting
      single-cell transcriptome'
  source_content_hash: sha256:eac5082736b578ba798d9e4bd6babe245190300f3f8c61ce58dfe7f78b283ab1
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
  - Miscell
  - Scanpy
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# integrated-gradient-attribution-scoring

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Identifies cluster-specific marker genes from a pretrained Miscell encoder by attaching a 1-layer linear classifier to the frozen encoder and computing integrated-gradient attribution scores that quantify each gene's influence on each cell cluster.

## When to use

Use when cluster-specific marker genes must be recovered from a trained self-supervised scRNA-seq encoder (e.g., Miscell latent features) and attributed to individual genes via an attribution method.

## When NOT to use

- The gene of interest is not among the highly variable genes, since Miscell's marker identification is limited to HVGs and may miss cluster-specific markers outside them
- The dataset is too small, because Miscell demands a large sample size to obtain good performance
- Batch effects have not been corrected beforehand, since Miscell does not internally include any batch-correction method

## Inputs

- Pretrained Miscell encoder checkpoint (e.g., checkpoint_0199.pth.tar)
- Normalized HVG expression matrix (e.g., x_new_scanpy_HVG.npz, 1686 input features)
- Cell cluster labels from DBSCAN on FIt-SNE features or scanpy Leiden clustering

## Outputs

- Gene-by-cluster integrated-gradient attribution score table (integrated_gradient_scores.csv)
- Ranked list of cluster-specific marker genes per cluster

## How to apply

Load the pretrained Miscell deep neural network encoder (e.g., a densenet11 checkpoint such as checkpoint_0199.pth.tar) and freeze its weights so the self-supervised latent representation is preserved. Attach a single-layer linear classifier on top of the frozen encoder and train only that classifier on the cell cluster labels. Compute integrated-gradient attribution scores through the frozen encoder + linear classifier to quantify the influence of each gene on each cluster's classification. Rank genes by attribution score within each cluster and extract the top-ranked cluster-specific markers; write the full gene-by-cluster score table to integrated_gradient_scores.csv. Note that attribution is only computed over highly variable genes used as model input (e.g., the HVG feature matrix x_new_scanpy_HVG.npz).

## Related tools

- **Miscell** (Self-supervised encoder whose frozen latent representation feeds the linear classifier used for integrated-gradient marker attribution) — https://github.com/lixiangchun/Miscell
- **Scanpy** (Clustering/neighborhood-graph framework whose Leiden cluster labels train the linear classifier and serve as reference marker calls)

## Examples

```
model = utils.load_pretrained_model(arch, in_features, checkpoint, return_feature=True); features = utils.extract_features(model, X)
```

## Evaluation signals

- Recovered CD4+ T cell markers include canonical CD4 and CD40LG plus CCR4, and CD8+ T cell markers include CD8A, CD8B, KLRK1 plus XCL1, matching Scanpy and Seurat calls
- Additional expected marker calls reproduced: SLAMF7/PRF1 (cluster C29), TIGIT/CCDC22 (C49), XCR1/SLC4A10/CTLA4 (C8), KLRG1 (C53)
- Encoder weights remain frozen throughout classifier training so the self-supervised latent representation is unchanged
- Full gene-by-cluster attribution table is written to integrated_gradient_scores.csv

## Limitations

- Marker identification is limited to the highly variable genes; cluster-specific marker genes outside HVGs may be missed
- Miscell demands a large sample size to obtain good performance
- Batch correction must be performed as a preprocessing step before Miscell; none is included internally
- Clinically significant latent features lack explainability and biological mechanisms underlying them remain unknown

## Evidence
