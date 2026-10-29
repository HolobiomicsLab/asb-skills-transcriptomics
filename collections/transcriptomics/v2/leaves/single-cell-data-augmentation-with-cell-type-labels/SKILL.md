---
name: single-cell-data-augmentation-with-cell-type-labels
description: Use when the number of paired training cells for scRNA-seq/scATAC-seq
  (or scADT-seq) translation is limited and clustering quality of translated profiles
  must be preserved, especially in five-fold down-sampling designs.
when_to_use_negative:
- Paired training cells are abundant (e.g., full 100% training set with large cell
  counts), where augmentation adds training time without clear benefit
- The task requires modalities scButterfly does not yet support, such as spatial transcriptomics
  or single-cell Hi-C data
- No paired multi-omics reference and no usable cell_type/cluster labels exist to
  drive the augmentation
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scButterfly
  role: Provides Butterfly.load_data, data_preprocessing, augmentation(aug_type),
    construct_model, train_model and test_model; aug_type='cell_type_augmentation'
    yields scButterfly-T, 'MultiVI_augmentation' yields scButterfly-C, and None yields
    the non-augmented scButterfly-B baseline
  repo: https://github.com/BioX-NKU/scButterfly
- name: MultiVI
  role: scvi-tools method used by the 'MultiVI_augmentation' path to generate cluster
    labels for synthetic samples when 'cell_type' is absent from adata.obs
provenance:
  source_task_ids:
  - task_003
  - task_004
  source_papers:
  - doi: 10.1038/s41467-024-47418-x
    title: 'scButterfly: a versatile single-cell cross-modality translation method
      via dual-aligned variational autoencoders'
  source_content_hash: sha256:fccefa82060d90546b7fecffef0632fca803c64959056f172cad723b078d4e15
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BioX-NKU/scButterfly
  tools:
  - MultiVI
  - Python
  - scButterfly
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 14
---

# single-cell-data-augmentation-with-cell-type-labels

## Summary

Data-augmentation scheme of scButterfly (scButterfly-T/-C) that expands the paired training set to roughly three times its size using cell-type labels or MultiVI-derived cluster labels, mitigating limited paired multi-omics cell numbers during cross-modality translation.

## When to use

Use when the number of paired training cells for scRNA-seq/scATAC-seq (or scADT-seq) translation is limited and clustering quality of translated profiles must be preserved, especially in five-fold down-sampling designs.

## When NOT to use

- Paired training cells are abundant (e.g., full 100% training set with large cell counts), where augmentation adds training time without clear benefit
- The task requires modalities scButterfly does not yet support, such as spatial transcriptomics or single-cell Hi-C data
- No paired multi-omics reference and no usable cell_type/cluster labels exist to drive the augmentation

## Inputs

- AnnData object of scRNA-seq raw counts (n_obs x n_vars, cells x genes)
- AnnData object of scATAC-seq raw counts (n_obs x n_vars, cells x peaks), peaks sorted by chromosome with chrom_list
- train_id / test_id / validation_id cell-ID lists from paired multi-omics data
- cell_type labels in adata.obs (for scButterfly-T) or a MultiVI-trained cluster labeling (for scButterfly-C)

## Outputs

- Augmented paired training data expanded to ~3x the subsampled size, consumed by Butterfly.construct_model/train_model
- Translated target-modality profiles from Butterfly.test_model (e.g., A2R_predict, R2A_predict)
- Clustering metrics (e.g., ARI/NMI via calculate_cluster.calculate_cluster_index) aggregated across folds and sampling fractions (e.g., figure_2d_clustering_metrics.csv)

## How to apply

After loading raw count matrices with Butterfly.load_data and preprocessing with Butterfly.data_preprocessing, call Butterfly.augmentation(aug_type) before model construction. Choose 'cell_type_augmentation' (scButterfly-T) when 'cell_type' is present in adata.obs; if it is absent, scButterfly automatically falls back to 'MultiVI_augmentation' (scButterfly-C), which first trains a MultiVI model (Leiden-based clustering via scvi-tools) to derive cluster labels. The augmented training data expands to about three times the original subsampled size, then proceeds to Butterfly.construct_model, Butterfly.train_model, and Butterfly.test_model. Rationale: synthetic samples built from cell-type/cluster identity improve translation performance and alleviate the restricted number of cells measured by multi-omics protocols, with the benefit being largest at low training fractions (20%–100% down-sampling).

## Related tools

- **scButterfly** (Provides Butterfly.load_data, data_preprocessing, augmentation(aug_type), construct_model, train_model and test_model; aug_type='cell_type_augmentation' yields scButterfly-T, 'MultiVI_augmentation' yields scButterfly-C, and None yields the non-augmented scButterfly-B baseline) — https://github.com/BioX-NKU/scButterfly
- **MultiVI** (scvi-tools method used by the 'MultiVI_augmentation' path to generate cluster labels for synthetic samples when 'cell_type' is absent from adata.obs)

## Examples

```
from scButterfly.butterfly import Butterfly; butterfly = Butterfly(); butterfly.load_data(RNA_data, ATAC_data, train_id, test_id); butterfly.data_preprocessing(); butterfly.augmentation(aug_type='cell_type_augmentation')
```

## Evaluation signals

- Augmented training set size equals approximately three times the subsampled training-cell count
- Clustering metrics (calculate_cluster_index) of scButterfly-T and scButterfly-C translated profiles exceed scButterfly-B at limited training fractions, matching the Fig. 2d ordering
- Mean metrics aggregated across five cross-validation folds and sampling fractions (20%-100%) are consistent in the output CSV (e.g., figure_2d_clustering_metrics.csv)
- t-SNE plots (draw_cluster.draw_tsne) of translated profiles show preserved cell-type separation relative to true labels
- Check that supplying 'cell_type' in adata.obs keeps the pipeline in scButterfly-T mode rather than the automatic MultiVI fallback

## Limitations

- Augmentation increases training time, though it promises better prediction results per the README
- If 'cell_type' is missing from adata.obs, scButterfly-T silently transfers to MultiVI_augmentation, so label provenance must be checked
- Performance gains are most evident when training cells are limited; the three-fold expansion ratio was confirmed optimal in the article's down-sampling experiments
- Cross-cell-type and cross-organ translation performance is flagged by the authors as still needing improvement

## Evidence

- [intro] Augmentation rationale for limited multi-omics cell numbers: "an effective data augmentation scheme may further improve the translation performance, alleviate the issues of limited number of cells measured by multi-omics protocols"
- [readme] augmentation API and aug_type choices: [evidence span withheld — anchor status: external_doc]
- [readme] scButterfly-T label lookup and automatic fallback: [evidence span withheld — anchor status: external_doc]
- [results] Augmentation benefit demonstrated under limited training cells: [evidence span withheld — anchor status: not_found]
