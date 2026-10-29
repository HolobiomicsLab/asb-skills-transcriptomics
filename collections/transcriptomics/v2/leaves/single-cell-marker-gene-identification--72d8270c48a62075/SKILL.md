---
name: single-cell-marker-gene-identification
description: Use when you have a trained Miscell encoder on scRNA-seq data (e.g.,
  Smart-seq HVG matrix) with cell cluster labels and need cluster-specific marker
  genes via attribution scores rather than differential expression tests.
when_to_use_negative:
- The genes you hope to recover fall outside the highly variable gene set — Miscell's
  marker identification is limited to HVGs, so cluster-specific markers outside HVGs
  may be missed.
- The dataset is small — Miscell demands a large sample size to obtain good performance.
- You need marker discovery from raw counts without a pretrained encoder — use differential
  expression (t test / Wilco rank-sum in Scanpy or Seurat) instead.
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Miscell
  role: Provides the self-supervised deep neural network encoder whose frozen features
    feed the 1-layer linear classifier; attribution scores quantify each gene's influence
    on cluster classification
  repo: https://github.com/lixiangchun/Miscell
- name: Scanpy
  role: Used for data preprocessing (HVG selection/normalization) and as a reference
    method whose differential analysis markers are compared to the attribution-based
    markers
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.isci.2021.103200
    title: 'Miscell: An efficient self-supervised learning approach for dissecting
      single-cell transcriptome'
  source_content_hash: sha256:d236e660c17ff2c82955710b0824c7ab47c837525c40971af56edc820b8befde
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
yes_rate: 0.8
n_claims: 10
---

# single-cell-marker-gene-identification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Identifies cluster-specific marker genes from single-cell RNA-seq data by quantifying each gene's influence on cell clusters through a trained Miscell encoder, using an attribution method (integrated gradients) on a frozen encoder plus a 1-layer linear classifier.

## When to use

Use when you have a trained Miscell encoder on scRNA-seq data (e.g., Smart-seq HVG matrix) with cell cluster labels and need cluster-specific marker genes via attribution scores rather than differential expression tests.

## When NOT to use

- The genes you hope to recover fall outside the highly variable gene set — Miscell's marker identification is limited to HVGs, so cluster-specific markers outside HVGs may be missed.
- The dataset is small — Miscell demands a large sample size to obtain good performance.
- You need marker discovery from raw counts without a pretrained encoder — use differential expression (t test / Wilco rank-sum in Scanpy or Seurat) instead.

## Inputs

- Pretrained Miscell model checkpoint (e.g. checkpoint_0199.pth.tar, densenet architecture)
- Normalized HVG expression matrix (.npz, e.g. data/x_new_scanpy_HVG.npz with in_features=1686)
- Cell cluster labels (e.g. from DBSCAN on FIt-SNE features or scanpy Leiden clustering)

## Outputs

- Gene-by-cluster integrated-gradient attribution score table (integrated_gradient_scores.csv)
- Top-ranked cluster-specific marker genes per cluster

## How to apply

Load the pretrained Miscell deep neural network encoder and freeze its weights to preserve the self-supervised latent representation, then attach a single-layer linear classifier on top and train only that classifier on the cell cluster labels. Compute integrated-gradient attribution scores quantifying each gene's influence on each cluster's classification through the frozen encoder + linear classifier, then rank genes by attribution score within each cluster to extract top-ranked markers. This replaces differential analysis such as the t test and Wilcoxon rank-sum test adopted by Seurat and Scanpy. On the Smart-seq dataset, the expected recovered markers are CD4/CD40LG/CCR4 for CD4+ T cells, CD8A/CD8B/KLRK1/XCL1 for CD8+ T cells, SLAMF7/PRF1 for cluster C29, TIGIT/CCDC22 for cluster C49, XCR1/SLC4A10/CTLA4 for cluster C8, and KLRG1 for cluster C53. Note that only highly variable genes (the HVG input, e.g. the 1686-feature x_new_scanpy_HVG.npz matrix) can receive attribution scores.

## Related tools

- **Miscell** (Provides the self-supervised deep neural network encoder whose frozen features feed the 1-layer linear classifier; attribution scores quantify each gene's influence on cluster classification) — https://github.com/lixiangchun/Miscell
- **Scanpy** (Used for data preprocessing (HVG selection/normalization) and as a reference method whose differential analysis markers are compared to the attribution-based markers)

## Examples

```
model = utils.load_pretrained_model(arch, in_features, checkpoint, return_feature=True); features = utils.extract_features(model, X)
```

## Evaluation signals

- Recovered markers reproduce the canonical calls: CD4, CD40LG, plus CCR4 for CD4+ T cells and CD8A, CD8B, KLRK1, plus XCL1 for CD8+ T cells on the Smart-seq dataset.
- Cluster-specific calls match reported results: SLAMF7/PRF1 (C29), TIGIT/CCDC22 (C49), XCR1/SLC4A10/CTLA4 (C8), KLRG1 (C53).
- Attribution markers agree with those found by Scanpy and Seurat differential analysis.
- The full gene-by-cluster attribution score table is written to integrated_gradient_scores.csv.
- Encoder weights remain frozen after classifier training (only the linear classifier is updated), so the self-supervised latent representation is preserved.

## Limitations

- Identification of cluster-specific marker genes by Miscell is limited to the highly variable genes; markers outside HVGs may be missed.
- Miscell demands a large sample size to obtain good performance.
- The clinically significant latent features extracted by Miscell lack explainability and future works are required to address this issue.

## Evidence
