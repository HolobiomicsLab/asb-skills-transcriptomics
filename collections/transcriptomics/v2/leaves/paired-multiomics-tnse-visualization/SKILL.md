---
name: paired-multiomics-tnse-visualization
description: Use when you need qualitative visual confirmation that translated single-cell
  profiles (predicted scRNA-seq or scATAC-seq from a cross-modality translator) cluster
  by cell type and match true labeled profiles.
when_to_use_negative:
- Input is raw un-translated single-modality data where no cross-modality translation
  quality needs assessment
- Quantitative clustering indices (e.g., ARI/NMI from calculate_cluster_index) already
  suffice and no qualitative figure is required
- Very small cell sets (e.g., a down-sampled fold with only tens of cells) where t-SNE
  embeddings are unstable and misleading
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scButterfly
  role: Provides the translated profiles via Butterfly.test_model and the draw_cluster.draw_tsne
    visualization API used to render t-SNE plots of translated vs. true profiles
  repo: https://github.com/BioX-NKU/scButterfly
- name: Python 3.9
  role: Runtime environment for scButterfly (conda create scButterfly python==3.9)
    in which draw_tsne and calculate_cluster_index are executed
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-024-47418-x
    title: 'scButterfly: a versatile single-cell cross-modality translation method
      via dual-aligned variational autoencoders'
  source_content_hash: sha256:df0791cf89564c59c83a1addd689d9ca1c8d94ec76d54575b5cb9693138036d6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BioX-NKU/scButterfly
  tools:
  - Python
  - Python 3.9
  - scButterfly
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# paired-multiomics-tnse-visualization

## Summary

Renders t-SNE plots of scButterfly-translated single-cell profiles (e.g., predicted scATAC from scRNA or vice versa) colored by cell-type labels, providing qualitative validation that translated cross-modality data preserves cellular heterogeneity.

## When to use

Use when you need qualitative visual confirmation that translated single-cell profiles (predicted scRNA-seq or scATAC-seq from a cross-modality translator) cluster by cell type and match true labeled profiles.

## When NOT to use

- Input is raw un-translated single-modality data where no cross-modality translation quality needs assessment
- Quantitative clustering indices (e.g., ARI/NMI from calculate_cluster_index) already suffice and no qualitative figure is required
- Very small cell sets (e.g., a down-sampled fold with only tens of cells) where t-SNE embeddings are unstable and misleading

## Inputs

- Translated/predicted AnnData profiles from Butterfly.test_model (e.g., A2R_predict, R2A_predict)
- True paired target-modality AnnData profiles (raw counts preprocessed via Butterfly.data_preprocessing)
- Cell-type labels (adata.obs 'cell_type') or MultiVI-derived cluster labels for coloring

## Outputs

- t-SNE scatter plots of translated scRNA-seq/scATAC-seq profiles colored by cell type (draw_cluster.draw_tsne output)
- Qualitative comparison panels supporting clustering-metric CSVs (e.g., figure_2d_clustering_metrics.csv)

## How to apply

After predicting the held-out target modality with Butterfly.test_model (e.g., R2A_predict for RNA-to-ATAC translation), render t-SNE embeddings of the translated profiles alongside true profiles using draw_cluster.draw_tsne, coloring cells by their true cell-type labels. In the data-augmentation benchmark, generate one set of t-SNE plots per scButterfly variant (B basic, T cell-type-augmented, C MultiVI-cluster-augmented) and per training fraction (20%–100% of paired MCC cells), so cluster tightness can be compared visually across conditions. The rationale is that augmentation may 'further improve the translation performance, alleviate the issues of limited number of cells'; t-SNE makes this visible as better-separated, label-consistent clusters for scButterfly-T and scButterfly-C over scButterfly-B at low cell fractions. Pair the visual readout with quantitative clustering metrics from calculate_cluster.calculate_cluster_index, which indexes translated profiles against true labels, so the figure is not judged on aesthetics alone.

## Related tools

- **scButterfly** (Provides the translated profiles via Butterfly.test_model and the draw_cluster.draw_tsne visualization API used to render t-SNE plots of translated vs. true profiles) — https://github.com/BioX-NKU/scButterfly
- **Python 3.9** (Runtime environment for scButterfly (conda create scButterfly python==3.9) in which draw_tsne and calculate_cluster_index are executed)

## Examples

```
A2R_predict, R2A_predict = butterly.test_model(); from scButterfly.draw_cluster import draw_tsne; draw_tsne(R2A_predict, ATAC_test_data, cell_type_labels)
```

## Evaluation signals

- t-SNE clusters of translated profiles co-localize with clusters of true profiles for the same cell types
- Translated-profile plots for scButterfly-T and scButterfly-C show visibly tighter, label-consistent clusters than scButterfly-B at limited training-cell fractions (20%–100% folds)
- Visual ordering matches the quantitative calculate_cluster_index metrics aggregated across folds and sampling fractions
- Plots are reproducible across the five cross-validation folds with consistent cell-type color mappings
- Predicted inputs to the plot are verifiably the outputs of Butterfly.test_model on held-out paired cells, not training cells

## Limitations

- t-SNE is qualitative; acceptance of the augmentation benefit must be backed by calculate_cluster_index metrics, not the embedding alone
- scButterfly-T requires 'cell_type' in adata.obs and automatically falls back to MultiVI_augmentation if labels are absent, changing the visual comparison condition
- Performance for cross-cell-type and cross-organ translation is flagged by the authors as needing improvement, which can make t-SNE clusters of translated profiles less separable in those regimes

## Evidence

- [intro] an effective data augmentation scheme may further improve the translation performance, alleviate the issues of limited number of cells measured by multi-omics protocols: "an effective data augmentation scheme may further improve the translation performance, alleviate the issues of limited number of cells measured by multi-omics protocols"
- [readme] You could choose parameter `aug_type` from `cell_type_augmentation` or `MultiVI_augmentation`, this will cause more training time used, but promise better result for predicting.: [evidence span withheld — anchor status: external_doc]
- [other] 7. Render t-SNE visualizations of translated profiles with draw_cluster.draw_tsne for qualitative comparison.: [evidence span withheld — anchor status: not_found]
