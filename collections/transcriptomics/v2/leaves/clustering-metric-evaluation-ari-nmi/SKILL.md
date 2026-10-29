---
name: clustering-metric-evaluation-ari-nmi
description: Use when comparing clustering quality of translated cross-modality profiles
  (e.g., scButterfly-B vs scButterfly-T vs scButterfly-C on MCC scRNA–scATAC data)
  against true cell-type labels to quantify translation fidelity.
when_to_use_negative:
- No ground-truth cell-type labels are available for the test cells, so clustering
  indices against true labels cannot be computed
- The task is modality-specific profile accuracy (e.g., gene expression correlation)
  rather than preservation of cellular heterogeneity/cluster structure
- Input is raw single-modality data that has not yet been translated by a cross-modality
  model
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scButterfly
  role: Generates translated single-modality profiles via test_model and provides
    calculate_cluster.calculate_cluster_index and draw_cluster.draw_tsne for clustering-quality
    scoring and visualization
  repo: https://github.com/BioX-NKU/scButterfly
- name: MultiVI (scvi-tools)
  role: Supplies cluster labels (Leiden + MultiVI integration) used in the scButterfly-C
    augmentation arm being compared under the clustering metrics
  repo: https://docs.scvi-tools.org/en/stable/tutorials/notebooks/MultiVI_tutorial.html
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-024-47418-x
    title: 'scButterfly: a versatile single-cell cross-modality translation method
      via dual-aligned variational autoencoders'
  source_content_hash: sha256:6b1773a719e5361251ece80ad3d21a94b1d7171ae6739f167ef3f9dbf4444024
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BioX-NKU/scButterfly
  tools:
  - MultiVI (scvi-tools)
  - Python
  - scButterfly
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# Clustering metric evaluation (ARI/NMI) of cross-modality translations

## Summary

Quantifies how faithfully translated single-cell profiles (e.g., scRNA-seq ↔ scATAC-seq translations from scButterfly-B/T/C) preserve cell-type structure by clustering the predicted profiles and scoring them against true labels with calculate_cluster.calculate_cluster_index, supplemented by t-SNE visualizations (draw_cluster.draw_tsne).

## When to use

Use when comparing clustering quality of translated cross-modality profiles (e.g., scButterfly-B vs scButterfly-T vs scButterfly-C on MCC scRNA–scATAC data) against true cell-type labels to quantify translation fidelity.

## When NOT to use

- No ground-truth cell-type labels are available for the test cells, so clustering indices against true labels cannot be computed
- The task is modality-specific profile accuracy (e.g., gene expression correlation) rather than preservation of cellular heterogeneity/cluster structure
- Input is raw single-modality data that has not yet been translated by a cross-modality model

## Inputs

- Predicted (translated) scRNA-seq or scATAC-seq profiles from Butterfly.test_model
- True cell-type labels for the held-out test cells
- Per-fold train/test cell ID splits over the MCC paired dataset
- Variant identifier (scButterfly-B, scButterfly-T, scButterfly-C) and sampling fraction

## Outputs

- Clustering metrics (e.g., ARI/NMI-style indices from calculate_cluster_index) per variant, fold, and sampling fraction
- Aggregated mean clustering metrics table, e.g., figure_2d_clustering_metrics.csv
- t-SNE plots of translated profiles colored by cluster (draw_cluster.draw_tsne)

## How to apply

Predict the held-out target modality with Butterfly.test_model, then cluster the predicted profiles and score them against ground-truth labels using scButterfly's calculate_cluster.calculate_cluster_index, rendering t-SNE maps with draw_cluster.draw_tsne for qualitative confirmation. In the down-sampling design that surfaced this skill, repeat evaluation across five cross-validation folds and training fractions from 20% to 100% of paired MCC cells, comparing scButterfly-B (no augmentation), scButterfly-T (cell-type-label augmentation) and scButterfly-C (MultiVI-derived cluster augmentation, expanding training data three-fold). Aggregate mean clustering metrics across folds and fractions per variant (saved as figure_2d_clustering_metrics.csv) to judge whether augmentation mitigates limited training-cell numbers. Correct application is confirmed when scButterfly-T and scButterfly-C metrics exceed scButterfly-B at limited training-cell fractions, reproducing the Fig. 2d ordering with the three-fold augmentation ratio as optimal.

## Related tools

- **scButterfly** (Generates translated single-modality profiles via test_model and provides calculate_cluster.calculate_cluster_index and draw_cluster.draw_tsne for clustering-quality scoring and visualization) — https://github.com/BioX-NKU/scButterfly
- **MultiVI (scvi-tools)** (Supplies cluster labels (Leiden + MultiVI integration) used in the scButterfly-C augmentation arm being compared under the clustering metrics) — https://docs.scvi-tools.org/en/stable/tutorials/notebooks/MultiVI_tutorial.html

## Examples

```
A2R_predict, R2A_predict = butterly.test_model()
```

## Evaluation signals

- scButterfly-T and scButterfly-C clustering metrics exceed scButterfly-B at limited training-cell fractions (20%–100%), reproducing the Fig. 2d ordering
- The three-fold augmentation ratio is confirmed as optimal across folds
- Mean clustering metrics aggregated over five cross-validation folds show consistent per-variant rankings (low fold-to-fold variance)
- t-SNE maps from draw_cluster.draw_tsne show cluster concordance with true labels matching the quantitative metrics

## Limitations

- scButterfly-C requires training a MultiVI model first, increasing evaluation runtime; README warns augmentation 'will cause more training time used'
- If 'cell_type' is absent from adata.obs, scButterfly-T automatically falls back to MultiVI-based augmentation, changing the variant being evaluated
- Evaluation depends on label quality; the article notes incorporation of heterogeneity information from annotated cell atlases remains a future improvement

## Evidence

- [readme] README augmentation basis: [evidence span withheld — anchor status: external_doc]
- [abstract] superiority finding: "we provide compelling evidence of scButterfly’s superiority over baseline methods in preserving cellular heterogeneity while translating datasets of various contexts"
- [intro] augmentation rationale: "an effective data augmentation scheme may further improve the translation performance, alleviate the issues of limited number of cells measured by multi-omics protocols"
