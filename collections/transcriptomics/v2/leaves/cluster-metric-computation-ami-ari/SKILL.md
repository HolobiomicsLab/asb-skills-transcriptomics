---
name: cluster-metric-computation-ami-ari
description: Use when you need to quantify how well predicted single-cell profiles
  (e.g., ADT from consecutive epigenome→transcriptome→proteome translation) preserve
  cell-type structure, comparing methods via AMI/ARI/HOM/NMI.
when_to_use_negative:
- You need to generate translated profiles itself — this skill only scores clustering
  concordance of already-predicted profiles, not the translation (use train_model/train_model_cite/Model.test
  first).
- Your evaluation target is per-feature reconstruction accuracy (e.g., correlation
  of predicted gene/protein values) rather than cell-type clustering concordance.
- Input profiles span novel cell types or cross-organ settings the article flags as
  needing improvement — clustering scores there may reflect known scButterfly weaknesses
  rather than pipeline errors.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_0091
tools:
- name: scButterfly
  role: Provides calculate_cluster.calculate_cluster_index for computing AMI/ARI/HOM/NMI
    and the chained translators whose predictions are scored
  repo: https://github.com/BioX-NKU/scButterfly
- name: scButterfly draw_cluster
  role: Renders t-SNE embeddings (draw_tsne) of predicted profiles to visually verify
    cell-type separation alongside the numeric cluster metrics
  repo: https://github.com/BioX-NKU/scButterfly
- name: BABEL
  role: Baseline whose predictions (combined with sciPENN) are scored with the same
    cluster metrics for comparison
- name: Polarbear
  role: Baseline whose predictions (combined with sciPENN) are scored with the same
    cluster metrics for comparison
- name: sciPENN
  role: Bridges baseline translator outputs to ADT predictions that are evaluated
    with the cluster metrics
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-024-47418-x
    title: 'scButterfly: a versatile single-cell cross-modality translation method
      via dual-aligned variational autoencoders'
  source_content_hash: sha256:7b71ebb0cd73630be8ec18666e4a90902a1bde7217970905e126d5b9e8a857b8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BioX-NKU/scButterfly
  tools:
  - BABEL
  - Polarbear
  - Python
  - scButterfly
  - scButterfly calculate_cluster
  - scButterfly draw_cluster
  - sciPENN
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# cluster-metric-computation-ami-ari

## Summary

Scores single-cell cross-modality translation quality by computing AMI, ARI, HOM, and NMI clustering concordance of predicted profiles (e.g., ADT translated from scATAC-seq via chained scButterfly translators) against ground-truth cell-type labels and baseline predictions.

## When to use

Use when you need to quantify how well predicted single-cell profiles (e.g., ADT from consecutive epigenome→transcriptome→proteome translation) preserve cell-type structure, comparing methods via AMI/ARI/HOM/NMI.

## When NOT to use

- You need to generate translated profiles itself — this skill only scores clustering concordance of already-predicted profiles, not the translation (use train_model/train_model_cite/Model.test first).
- Your evaluation target is per-feature reconstruction accuracy (e.g., correlation of predicted gene/protein values) rather than cell-type clustering concordance.
- Input profiles span novel cell types or cross-organ settings the article flags as needing improvement — clustering scores there may reflect known scButterfly weaknesses rather than pipeline errors.

## Inputs

- Raw count matrices as AnnData objects (BMMC batch scATAC-seq/scRNA-seq; CITE_BMMC paired scRNA-seq/ADT)
- Predicted ADT profiles from chained scButterfly Model.test inference
- Ground-truth BMMC cell-type label annotations
- Baseline predicted profiles from BABEL+sciPENN and Polarbear+sciPENN

## Outputs

- AMI, ARI, HOM, and NMI clustering scores
- t-SNE embeddings of predicted profiles (draw_cluster.draw_tsne)
- Comparison table consecutive_translation_metrics.tsv

## How to apply

First run the chained translation: train the epigenome-to-transcriptome translator on BMMC batches (scButterfly.train_model.Model.train with dual-aligned VAEs) and the transcriptome-to-proteome translator on CITE_BMMC (scButterfly.train_model_cite.Model.train), then apply the first Model.test followed by the CITE translator's Model.test to obtain predicted ADT profiles. Preprocess with RNA_data_preprocessing, ATAC_data_preprocessing, and CLR_transform so predicted and measured profiles live in comparable spaces. Call calculate_cluster.calculate_cluster_index to compute AMI, ARI, HOM, and NMI between clusters of the predicted profiles and ground-truth BMMC cell-type labels, and render t-SNE embeddings with draw_cluster.draw_tsne for visual verification of nuanced cell-type separation. Repeat the identical procedure on baseline predictions from BABEL+sciPENN and Polarbear+sciPENN and write all scores to consecutive_translation_metrics.tsv. The rationale: higher clustering concordance demonstrates the consecutive translation resolves cell-type differences better than single-modality baseline combinations.

## Related tools

- **scButterfly** (Provides calculate_cluster.calculate_cluster_index for computing AMI/ARI/HOM/NMI and the chained translators whose predictions are scored) — https://github.com/BioX-NKU/scButterfly
- **scButterfly draw_cluster** (Renders t-SNE embeddings (draw_tsne) of predicted profiles to visually verify cell-type separation alongside the numeric cluster metrics) — https://github.com/BioX-NKU/scButterfly
- **BABEL** (Baseline whose predictions (combined with sciPENN) are scored with the same cluster metrics for comparison)
- **Polarbear** (Baseline whose predictions (combined with sciPENN) are scored with the same cluster metrics for comparison)
- **sciPENN** (Bridges baseline translator outputs to ADT predictions that are evaluated with the cluster metrics)

## Examples

```
from scButterfly.calculate_cluster import calculate_cluster_index; calculate_cluster_index(pred_adata, true_adata)
```

## Evaluation signals

- All four metrics (AMI, ARI, HOM, NMI) are reported in consecutive_translation_metrics.tsv for scButterfly and both baseline combinations, computed by calculate_cluster.calculate_cluster_index.
- Scores are computed against ground-truth BMMC cell-type labels, not against a model's own clusters.
- scButterfly's consecutive-translation clustering scores should exceed those of BABEL+sciPENN and Polarbear+sciPENN, per the paper's finding of overall superior cell clustering performance.
- t-SNE embeddings from draw_cluster.draw_tsne should show predicted-ADT clusters aligning with the numeric metric ordering and resolving nuanced differences among cells.
- Identical preprocessing (RNA_data_preprocessing, ATAC_data_preprocessing, CLR_transform) is applied before metric computation so predicted and measured profiles are comparable.

## Limitations

- The article flags performance for cross-cell-type and cross-organ translation as needing improvement, so cluster metrics computed in those regimes may be pessimistic.
- The benchmarked baselines are limited to BABEL+sciPENN and Polarbear+sciPENN; other translators are not covered by this comparison.
- Skipping the data augmentation API (scButterfly-T/C) saves training time but degrades prediction quality, which propagates to the clustering scores.

## Evidence

- [abstract] abstract: "scButterfly can be generalized to unpaired data training, perturbation-response analysis, and consecutive translation."
- [methods] methods: "Translation between scRNA-seq and scADT-seq paired data"
- [readme] readme: [evidence span withheld — anchor status: external_doc]
