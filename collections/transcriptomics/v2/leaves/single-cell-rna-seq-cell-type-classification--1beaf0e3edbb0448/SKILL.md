---
name: single-cell-rna-seq-cell-type-classification
description: Use when training or evaluating cross-tissue scRNA-seq cell type classifiers
  on the CELLxGENE census (e.g., scaling 1.7M vs 16.2M-parameter scTab models) and
  reporting macro F1-score on held-out donors.
when_to_use_negative:
- The task requires very fine-grained or tissue-specific subtypes that the Cell Ontology
  cannot resolve (e.g., brain cell types where current ontologies lack resolution),
  since subtype-based scoring will misclassify ontology gaps as errors
- The data does not meet the corpus filters (fewer than 5000 cells per cell type,
  fewer than 30 donors per type, non-primary data, or assay outside PROTOCOLS), so
  donor-based generalization cannot be reliably quantified
- The input is multimodal (protein, spatial, chromatin) — scTab in this work uses
  only transcriptomic tabular input of 19,331 protein-coding genes
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: scTab
  role: The TabNet-based cell type classification model being instantiated, trained,
    and scaled; checkpoints and training data are distributed via theislab/scTab
  repo: https://github.com/theislab/scTab
- name: TabNet
  role: Tabular-data architecture selected for scTab, used to build the 1.7M and 16.2M
    parameter model configurations
- name: cellxgene-census
  role: Standardized Python API to query and download the CELLxGENE (CxG) census corpus
    for training and test data
  repo: https://chanzuckerberg.github.io/cellxgene-census/python-api.html
- name: NVIDIA Merlin dataloader
  role: GPU-optimized on-disk dataloader used to feed the large cell-by-gene training
    corpus to PyTorch during training
  repo: https://github.com/NVIDIA-Merlin/dataloader
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-024-51059-5
    title: 'scTab: Scaling cross-tissue single-cell annotation models'
  source_content_hash: sha256:032d847ce2d402b3f7ae407ce6390a46d59ec6272520a0ac1e3dbd908fea7ab7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/theislab/scTab
  tools:
  - NVIDIA Merlin dataloader
  - TabNet
  - cellxgene-census
  - scTab
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9375
n_claims: 16
---

# single-cell-rna-seq-cell-type-classification

## Summary

Train and evaluate the scTab (TabNet-based) cross-tissue cell type classifier on the CELLxGENE census corpus (v2023-05-15), including model-size scaling from ~1.7M to ~16.2M parameters. Performance is measured by macro F1-score on a donor-held-out test set, where a prediction counts as correct when a subtype of the true cell type is predicted.

## When to use

Use when training or evaluating cross-tissue scRNA-seq cell type classifiers on the CELLxGENE census (e.g., scaling 1.7M vs 16.2M-parameter scTab models) and reporting macro F1-score on held-out donors.

## When NOT to use

- The task requires very fine-grained or tissue-specific subtypes that the Cell Ontology cannot resolve (e.g., brain cell types where current ontologies lack resolution), since subtype-based scoring will misclassify ontology gaps as errors
- The data does not meet the corpus filters (fewer than 5000 cells per cell type, fewer than 30 donors per type, non-primary data, or assay outside PROTOCOLS), so donor-based generalization cannot be reliably quantified
- The input is multimodal (protein, spatial, chromatin) — scTab in this work uses only transcriptomic tabular input of 19,331 protein-coding genes

## Inputs

- CELLxGENE census (v2023-05-15) cell-by-gene matrix, donor-based train/val/test split, 15,240,192 training cells over 19,331 protein-coding genes and 164 cell types
- Cell Ontology child-node lookup matrix (04_create_hierarchy_matrices.ipynb) for subtype-aware prediction scoring
- Data stored in on-disk Merlin dataloader format (merlin_cxg_2023_05_15_sf-log1p, 164GB tarball)
- scTab model configuration specifying smallest (~1.7M parameters) and largest (~16.2M parameters) TabNet variants

## Outputs

- Per-configuration macro F1-scores on the donor-held-out test set (3,448,832 cells), e.g., scTab_model_scaling_results.csv
- Trained scTab (TabNet) classifier checkpoints for 1.7M and 16.2M parameter configurations
- Cell type predictions with subtype-credit evaluation against Cell Ontology child nodes

## How to apply

Load the donor-based train split of the processed CELLxGENE census (v2023-05-15) built under the paper's filters (is_primary_data == True; assay in PROTOCOLS; cell types that are subtypes of 'native cell' with ≥5000 cells each, observed in ≥30 donors and ≥7 Cell Ontology parent nodes), yielding 15,240,192 cells over 19,331 protein-coding genes and 164 cell types. Normalize each cell to 10,000 counts (size-factor normalization) and log1p transform, then store in on-disk format for the NVIDIA Merlin dataloader. Instantiate scTab (TabNet architecture) with the smallest (~1.7M parameters) and largest (~16.2M parameters) configurations and train both with cross-entropy loss under identical preprocessing and donor-held-out training data. Evaluate both models on the held-out-donor test set (3,448,832 cells) using macro F1-score, crediting a prediction when the classifier outputs a subtype of the true cell type label via the Cell Ontology child-node lookup matrix. Write the per-configuration macro F1-scores (e.g., to scTab_model_scaling_results.csv) and compare against the reported 0.7864±0.0010 (smallest) and 0.8323±0.0010 (largest) within their reported uncertainty.

## Related tools

- **scTab** (The TabNet-based cell type classification model being instantiated, trained, and scaled; checkpoints and training data are distributed via theislab/scTab) — https://github.com/theislab/scTab
- **TabNet** (Tabular-data architecture selected for scTab, used to build the 1.7M and 16.2M parameter model configurations)
- **cellxgene-census** (Standardized Python API to query and download the CELLxGENE (CxG) census corpus for training and test data) — https://chanzuckerberg.github.io/cellxgene-census/python-api.html
- **NVIDIA Merlin dataloader** (GPU-optimized on-disk dataloader used to feed the large cell-by-gene training corpus to PyTorch during training) — https://github.com/NVIDIA-Merlin/dataloader

## Examples

```
import merlin.io; dataset = merlin.io.Dataset(PARQUET_FILE_PATHS, engine="parquet")
```

## Evaluation signals

- Reproduced macro F1-score for the ~1.7M-parameter model matches 0.7864±0.0010 within reported uncertainty on the donor-held-out test set
- Reproduced macro F1-score for the ~16.2M-parameter model matches 0.8323±0.0010 within reported uncertainty
- Dataset composition invariants: 15,240,192 train cells, 19,331 protein-coding genes, 164 cell types; test set of 3,448,832 cells from held-out donors
- Preprocessing invariant: each cell normalized to 10,000 counts followed by log1p transformation before training
- Correctness is counted when the classifier predicts a subtype of the true cell type label via the Cell Ontology child-node lookup matrix, not exact-label equality

## Limitations

- The Cell Ontology is incomplete: not all cell types (especially rare ones) can be matched to a corresponding ontology term, and ontology relationships are still under active discussion, affecting classification metrics
- Pre-trained scTab models still contain fairly coarsely annotated cell type labels, and annotation granularity is inconsistent across CELLxGENE datasets — issues data curation, not model architecture, must fix
- Plain macro F1-scores may not properly reflect the complexity of unbalanced datasets imbalanced across cell types, tissues, and sequencing protocols
- The benchmark excludes uncommon assay technologies due to the ≥5000-cells-per-type and ≥30-donors filters

## Evidence

- [methods] normalization: "1. Normalize each cell to have `10000` counts (size factor normalization)"
- [methods] donor filter: "5. Each cell type has to be observed in at least `30` donors to reliably quantify whether the classifier can 
  generalize to new donors."
- [abstract] scaling finding: "the performance of scTab scales both in terms of training dataset size and model size"
- [discussion] ontology caveat: "not all cell types can be correctly matched to a corresponding ontology term, this is especially a problem for rare cell types."
- [discussion] metric caveat: "plain macro F1-scores may not properly reflect the complexity of unbalanced datasets which are not just imbalanced concerning the distribution of cell types"
- [readme] training data: "Training data (compatible with Merlin Dataloader infrastructure): https://pklab.med.harvard.edu/felix/data/merlin_cxg_2023_05_15_sf-log1p.tar.gz (164GB)"
