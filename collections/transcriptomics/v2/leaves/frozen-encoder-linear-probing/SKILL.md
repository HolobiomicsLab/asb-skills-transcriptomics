---
name: frozen-encoder-linear-probing
description: Use when a trained self-supervised scRNA-seq encoder exists and cluster-specific
  marker genes must be recovered by quantifying per-gene influence on cluster predictions
  rather than by differential-expression testing.
when_to_use_negative:
- Markers are expected outside the highly variable gene set — Miscell's attribution-based
  marker identification is limited to HVGs, so non-HVG markers will be missed
- The dataset is small — Miscell demands a large sample size to obtain good performance
- Input data has uncorrected batch effects — batch correction must be done as a preprocessing
  step before Miscell
- The goal is a differential-expression p-value ranking rather than an attribution-based
  influence score
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Miscell
  role: Provides the frozen self-supervised deep neural network encoder whose latent
    features and attribution pathway are used for marker gene identification
  repo: github.com/lixiangchun/Miscell
- name: Scanpy
  role: Used to cluster the Miscell features (neighbors graph with n_neighbors=10
    on 'X' rep, Leiden clustering) to supply cluster labels for the linear probe and
    as the reference method the recovered markers are compared against
- name: FIt-SNE
  role: Performs t-SNE embedding of the extracted Miscell features (perplexity_list=[30,
    36, 42, 48]) prior to DBSCAN clustering
  repo: github.com/KlugerLab/FIt-SNE
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.isci.2021.103200
    title: 'Miscell: An efficient self-supervised learning approach for dissecting
      single-cell transcriptome'
  source_content_hash: sha256:7de4ff3e5652cadf34e5ee8964f437b4d59b25bbb7983d9908e74fe083c49e6f
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/lixiangchun/Miscell
  tools:
  - FIt-SNE
  - Miscell
  - Scanpy
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# frozen-encoder-linear-probing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Attaches a single-layer linear classifier to a pretrained Miscell self-supervised encoder with frozen weights, then applies integrated-gradient attribution through the frozen encoder + linear classifier to quantify each gene's influence on cell clusters and identify cluster-specific marker genes.

## When to use

Use when a trained self-supervised scRNA-seq encoder exists and cluster-specific marker genes must be recovered by quantifying per-gene influence on cluster predictions rather than by differential-expression testing.

## When NOT to use

- Markers are expected outside the highly variable gene set — Miscell's attribution-based marker identification is limited to HVGs, so non-HVG markers will be missed
- The dataset is small — Miscell demands a large sample size to obtain good performance
- Input data has uncorrected batch effects — batch correction must be done as a preprocessing step before Miscell
- The goal is a differential-expression p-value ranking rather than an attribution-based influence score

## Inputs

- Pretrained Miscell encoder checkpoint (e.g. checkpoint_0199.pth.tar, densenet architecture)
- Normalized gene expression matrix restricted to highly variable genes (data/x_new_scanpy_HVG.npz)
- Cell cluster labels (e.g. Leiden clusters from scanpy sc.tl.leiden on the Miscell features)

## Outputs

- Gene-by-cluster integrated-gradient attribution score table (integrated_gradient_scores.csv)
- Ranked list of top cluster-specific marker genes per cluster (e.g. CD4/CD40LG/CCR4 for CD4+ T cells, CD8A/CD8B/KLRK1/XCL1 for CD8+ T cells)

## How to apply

Load the pretrained Miscell encoder (e.g. a densenet checkpoint such as checkpoint_0199.pth.tar trained on data/x_new_scanpy_HVG.npz) and freeze its weights so the self-supervised latent representation is preserved. Train only a 1-layer linear classifier on top of the frozen encoder using the cell cluster labels, then compute integrated-gradient attribution scores that quantify the influence of each gene on each cluster's classification through the frozen encoder + linear classifier. Rank genes by attribution score within each cluster and take the top-ranked cluster-specific markers, checking them against canonical calls (CD4/CD40LG/CCR4 for CD4+ T cells; CD8A/CD8B/KLRK1/XCL1 for CD8+ T cells; SLAMF7/PRF1 for C29; TIGIT/CCDC22 for C49; XCR1/SLC4A10/CTLA4 for C8; KLRG1 for C53 on the Smart-seq dataset). Write the full gene-by-cluster attribution score table to integrated_gradient_scores.csv. Rationale: Miscell identifies markers via attribution instead of the t-test/Wilcoxon rank-sum differential analysis used by Seurat and Scanpy.

## Related tools

- **Miscell** (Provides the frozen self-supervised deep neural network encoder whose latent features and attribution pathway are used for marker gene identification) — github.com/lixiangchun/Miscell
- **Scanpy** (Used to cluster the Miscell features (neighbors graph with n_neighbors=10 on 'X' rep, Leiden clustering) to supply cluster labels for the linear probe and as the reference method the recovered markers are compared against)
- **FIt-SNE** (Performs t-SNE embedding of the extracted Miscell features (perplexity_list=[30, 36, 42, 48]) prior to DBSCAN clustering) — github.com/KlugerLab/FIt-SNE

## Examples

```
model = utils.load_pretrained_model(arch, in_features, checkpoint, return_feature=True); features = utils.extract_features(model, X)
```

## Evaluation signals

- On the Smart-seq dataset the top-ranked markers reproduce the canonical calls: CD4, CD40LG, plus CCR4 for CD4+ T cells and CD8A, CD8B, KLRK1, plus XCL1 for CD8+ T cells
- Cluster-specific markers for other clusters match reported calls: SLAMF7/PRF1 (C29), TIGIT/CCDC22 (C49), XCR1/SLC4A10/CTLA4 (C8), KLRG1 (C53)
- Recovered markers agree with those found by Scanpy and Seurat differential analysis (t-test/Wilcoxon rank-sum)
- integrated_gradient_scores.csv exists and contains a full gene-by-cluster attribution score table over the frozen encoder + linear classifier

## Limitations

- Cluster-specific marker gene identification by Miscell is limited to the highly variable genes; markers outside HVGs may be missed
- Miscell demands a large sample size to obtain good performance, analogous to other deep neural network methods
- Miscell does not internally include batch-correction methods; users must correct for batch effects before running it
- The clinically significant latent features extracted by Miscell lack explainability, requiring future work

## Evidence

- [results] attribution-based marker identification on Smart-seq dataset: [evidence span withheld — anchor status: not_found]
