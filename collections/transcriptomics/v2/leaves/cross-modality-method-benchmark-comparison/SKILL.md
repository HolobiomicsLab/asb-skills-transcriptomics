---
name: cross-modality-method-benchmark-comparison
description: Use when a single-modality dataset (e.g., scATAC-seq from BMMC batches)
  must be translated consecutively to a downstream modality (ADT via CITE_BMMC) and
  compared head-to-head with alternative translator pipelines under a labeled ground
  truth.
when_to_use_negative:
- The target modality is not supported by scButterfly or the baselines, e.g., spatial
  transcriptomics or single-cell Hi-C data, which the article names only as future
  directions
- The data to be translated spans cross-cell-type or cross-organ contexts, where the
  article flags scButterfly performance as still needing improvement via adaptive
  and interpretable model parameters
- No ground-truth cell-type labels or baseline predictions are available, so comparative
  clustering indices (AMI/ARI/HOM/NMI) cannot be computed
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scButterfly
  role: Core translator trained with dual-aligned VAEs; chained Model.test calls produce
    the predicted scRNA and ADT profiles under evaluation
  repo: https://github.com/BioX-NKU/scButterfly
- name: BABEL
  role: 'Baseline component: autoencoder-based ATAC↔RNA translator whose RNA predictions
    feed sciPENN for ADT'
- name: Polarbear
  role: 'Baseline component: VAE-stitching ATAC↔RNA translator whose predictions feed
    sciPENN for ADT'
- name: sciPENN
  role: 'Baseline component: recurrent-network RNA→ADT predictor appended to BABEL
    or Polarbear outputs'
- name: scButterfly calculate_cluster
  role: Computes AMI, ARI, HOM, NMI indices against ground-truth labels via calculate_cluster_index
  repo: https://github.com/BioX-NKU/scButterfly
- name: scButterfly draw_cluster
  role: Renders t-SNE embeddings of predicted profiles via draw_tsne for qualitative
    comparison
  repo: https://github.com/BioX-NKU/scButterfly
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-024-47418-x
    title: 'scButterfly: a versatile single-cell cross-modality translation method
      via dual-aligned variational autoencoders'
  source_content_hash: sha256:0b1cc785ed3afff4f77508f69126311f23fa5e0ed092ca59b0f39a9db907e719
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# cross-modality-method-benchmark-comparison

## Summary

Benchmarks scButterfly's chained cross-modality translation (scATAC-seq → scRNA-seq → ADT) against combination baselines (BABEL+sciPENN, Polarbear+sciPENN) on BMMC and CITE_BMMC data, using cell-type clustering scores (AMI, ARI, HOM, NMI) as the comparative yardstick.

## When to use

Use when a single-modality dataset (e.g., scATAC-seq from BMMC batches) must be translated consecutively to a downstream modality (ADT via CITE_BMMC) and compared head-to-head with alternative translator pipelines under a labeled ground truth.

## When NOT to use

- The target modality is not supported by scButterfly or the baselines, e.g., spatial transcriptomics or single-cell Hi-C data, which the article names only as future directions
- The data to be translated spans cross-cell-type or cross-organ contexts, where the article flags scButterfly performance as still needing improvement via adaptive and interpretable model parameters
- No ground-truth cell-type labels or baseline predictions are available, so comparative clustering indices (AMI/ARI/HOM/NMI) cannot be computed

## Inputs

- BMMC batch scATAC-seq/scRNA-seq raw count matrices (AnnData)
- CITE_BMMC paired scRNA-seq/ADT raw count matrices (AnnData)
- Ground-truth BMMC cell-type labels
- Predictions from baseline pipelines (BABEL+sciPENN, Polarbear+sciPENN)

## Outputs

- Consecutively predicted ADT profiles from scButterfly and each baseline
- AMI, ARI, HOM, NMI clustering score table (consecutive_translation_metrics.tsv)
- Differential expression of proteins (DEP) analysis of predicted vs. measured ADT
- t-SNE embeddings of predicted profiles (draw_cluster.draw_tsne)

## How to apply

Load the BMMC batch scATAC-seq/scRNA-seq data and CITE_BMMC paired scRNA-seq/ADT data with butterfly.Butterfly.load_data, preprocessing via data_processing.RNA_data_preprocessing, ATAC_data_preprocessing, and CLR_transform. Train the epigenome-to-transcriptome translator with scButterfly.train_model.Model.train using dual-aligned VAEs on BMMC batches, then train the transcriptome-to-proteome translator with scButterfly.train_model_cite.Model.train on CITE_BMMC. Run chained inference: feed the first translator's Model.test predictions into the CITE translator's Model.test to obtain predicted ADT profiles. Reproduce the same chained task with the combination baselines (BABEL+sciPENN, Polarbear+sciPENN) and compute AMI, ARI, HOM, and NMI with calculate_cluster.calculate_cluster_index against ground-truth BMMC cell-type labels, supporting the comparison with differential expression of proteins (DEP) analysis on predicted vs. measured ADT values and t-SNE embeddings via draw_cluster.draw_tsne. Write the score table to consecutive_translation_metrics.tsv and judge superiority by higher clustering indices plus clearer resolution of nuanced cell differences in the predicted ADT profiles.

## Related tools

- **scButterfly** (Core translator trained with dual-aligned VAEs; chained Model.test calls produce the predicted scRNA and ADT profiles under evaluation) — https://github.com/BioX-NKU/scButterfly
- **BABEL** (Baseline component: autoencoder-based ATAC↔RNA translator whose RNA predictions feed sciPENN for ADT)
- **Polarbear** (Baseline component: VAE-stitching ATAC↔RNA translator whose predictions feed sciPENN for ADT)
- **sciPENN** (Baseline component: recurrent-network RNA→ADT predictor appended to BABEL or Polarbear outputs)
- **scButterfly calculate_cluster** (Computes AMI, ARI, HOM, NMI indices against ground-truth labels via calculate_cluster_index) — https://github.com/BioX-NKU/scButterfly
- **scButterfly draw_cluster** (Renders t-SNE embeddings of predicted profiles via draw_tsne for qualitative comparison) — https://github.com/BioX-NKU/scButterfly

## Examples

```
from scButterfly.butterfly import Butterfly; butterfly = Butterfly(); butterfly.load_data(RNA_data, ATAC_data, train_id, test_id, validation_id)
```

## Evaluation signals

- AMI, ARI, HOM, and NMI values from calculate_cluster_index are computed against the same ground-truth BMMC cell-type labels for scButterfly and all baselines
- scButterfly's consecutive translation yields higher clustering indices than BABEL+sciPENN and Polarbear+sciPENN
- Comparison metrics are written to consecutive_translation_metrics.tsv with all methods present
- Predicted ADT profiles support DEP analysis against measured ADT values with coherent cell-type-specific differences
- t-SNE embeddings from draw_tsne resolve nuanced differences among cell types in the predicted ADT profiles

## Limitations

- Performance for cross-cell-type and cross-organ translation is flagged as needing improvement via advanced machine learning techniques with adaptive and interpretable model parameters
- Extension to additional modalities such as spatial transcriptomics and single-cell Hi-C data is a future direction, not currently supported
- Data augmentation via cell_type_augmentation or MultiVI_augmentation increases training time (though it promises better prediction), and scButterfly-T automatically falls back to MultiVI augmentation if cell_type is absent from adata.obs

## Evidence

- [abstract] scButterfly generalizes to consecutive translation as a benchmarked capability: "scButterfly can be generalized to unpaired data training, perturbation-response analysis, and consecutive translation."
- [abstract] scButterfly superiority claim grounding the benchmark outcome: "we provide compelling evidence of scButterfly’s superiority over baseline methods in preserving cellular heterogeneity while translating datasets of various contexts"
- [intro] Polarbear baseline mechanism: "Polarbear first trains a variational autoencoder (VAE) for each data modality and then stitches the frozen encoder of one modality with the frozen decoder"
- [readme] Data loading entry point for the benchmark pipeline: [evidence span withheld — anchor status: external_doc]
- [intro] Baseline sciPENN mechanism: "sciPENN integrates a sequence of feed-forward blocks with a recurrent neural network and performs end-to-end training"
