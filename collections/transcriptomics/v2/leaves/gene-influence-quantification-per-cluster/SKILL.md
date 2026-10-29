---
name: gene-influence-quantification-per-cluster
description: Use when marker genes must be identified for clusters from pretrained
  deep learning embeddings (e.g., Miscell) and you need per-gene, per-cluster influence
  scores rather than t-test/Wilcoxon differential expression.
when_to_use_negative:
- Marker genes outside the highly variable gene set are of interest — the article
  states 'cluster-specific marker genes may be missed in HVGs'
- Small datasets — Miscell 'demands a large sample size to obtain good performance'
- Input lacks batch correction and you expect the method to handle it — batch correction
  must be done as preprocessing before Miscell
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Miscell
  role: Provides the frozen deep neural network encoder whose latent features feed
    the linear classifier used for attribution-based marker identification
  repo: https://github.com/lixiangchun/Miscell
- name: Scanpy
  role: Used for neighborhood graph construction and Leiden clustering that define
    cluster labels, and as the differential-expression baseline (t-test/Wilcoxon)
    against which attribution-recovered markers are compared
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.isci.2021.103200
    title: 'Miscell: An efficient self-supervised learning approach for dissecting
      single-cell transcriptome'
  source_content_hash: sha256:81699f93afc516dee3c7ca0e2bf555d18d3ec694ebaea8146f874fb3a99752ea
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
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# gene-influence-quantification-per-cluster

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantifies each gene's influence on cell-cluster assignments by attaching a 1-layer linear classifier to a frozen Miscell encoder and applying integrated-gradient attribution, yielding cluster-specific marker genes for single-cell transcriptome data.

## When to use

Use when marker genes must be identified for clusters from pretrained deep learning embeddings (e.g., Miscell) and you need per-gene, per-cluster influence scores rather than t-test/Wilcoxon differential expression.

## When NOT to use

- Marker genes outside the highly variable gene set are of interest — the article states 'cluster-specific marker genes may be missed in HVGs'
- Small datasets — Miscell 'demands a large sample size to obtain good performance'
- Input lacks batch correction and you expect the method to handle it — batch correction must be done as preprocessing before Miscell

## Inputs

- Pretrained Miscell checkpoint (e.g., checkpoint_0199.pth.tar)
- Normalized HVG expression matrix (npz, e.g., data/x_new_scanpy_HVG.npz with in_features=1686)
- Cell cluster labels (from DBSCAN on FIt-SNE features or scanpy Leiden clustering)

## Outputs

- Gene-by-cluster integrated-gradient attribution score table (integrated_gradient_scores.csv)
- Ranked cluster-specific marker genes per cluster

## How to apply

Load the pretrained Miscell encoder (densenet11/21/29/63 trained with MoCo-style contrastive learning) and freeze its weights so the self-supervised latent representation is preserved. Attach a single-layer linear classifier on top of the frozen encoder and train only the classifier on the cell cluster labels derived from DBSCAN on FIt-SNE features or Leiden clustering. Compute integrated-gradient attribution scores quantifying each gene's influence on each cluster's classification through the frozen encoder + linear classifier, then rank genes by attribution score within each cluster and extract the top-ranked markers. Write the full gene-by-cluster score table to integrated_gradient_scores.csv. Note that only highly variable genes (HVGs) are scored, since the input feature matrix (e.g., x_new_scanpy_HVG.npz) is restricted to HVGs.

## Related tools

- **Miscell** (Provides the frozen deep neural network encoder whose latent features feed the linear classifier used for attribution-based marker identification) — https://github.com/lixiangchun/Miscell
- **Scanpy** (Used for neighborhood graph construction and Leiden clustering that define cluster labels, and as the differential-expression baseline (t-test/Wilcoxon) against which attribution-recovered markers are compared)

## Examples

```
model = utils.load_pretrained_model(arch, in_features, 'checkpoint_0199.pth.tar', return_feature=True); features = utils.extract_features(model, X)
```

## Evaluation signals

- On the Smart-seq dataset, CD4+ T cell markers CD4, CD40LG, plus CCR4 are recovered among top-ranked genes for the CD4+ T cluster
- CD8+ T cell markers CD8A, CD8B, KLRK1, plus XCL1 are recovered for the CD8+ T cluster
- Other cluster-specific markers reproduced: SLAMF7/PRF1 (C29), TIGIT/CCDC22 (C49), XCR1/SLC4A10/CTLA4 (C8), KLRG1 (C53)
- Recovered markers agree with the Scanpy and Seurat differential-analysis calls (t test / Wilcoxon rank-sum)
- integrated_gradient_scores.csv contains a score for every HVG (e.g., 1686 genes) per cluster with no missing entries

## Limitations

- Marker identification is limited to highly variable genes, so cluster-specific markers outside HVGs may be missed
- Miscell demands a large sample size to obtain good performance
- Miscell does not internally include any batch-correction methods; batch effects must be corrected before running
- The clinically significant latent features extracted by Miscell lack explainability

## Evidence
