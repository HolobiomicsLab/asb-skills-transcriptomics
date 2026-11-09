---
name: single-cell-donor-based-data-splitting
description: Use when training or evaluating cross-tissue cell type classifiers on
  multi-donor scRNA-seq corpora where test cells must come from donors never seen
  during training.
when_to_use_negative:
- When the research question is within-donor performance (e.g., annotating new cells
  from the same donors), a cell-level random split is appropriate and a donor split
  would underestimate attainable accuracy.
- When the corpus lacks reliable donor/batch metadata or cell types are observed in
  fewer than 30 donors, held-out-donor generalization cannot be reliably quantified.
- When data is not primary (is_primary_data == False), donor-based splitting cannot
  prevent train/test label leakage from duplicated observations.
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: scTab
  role: Training pipeline and repository containing the donor-based train/val/test
    split notebook (02_create_train_val_test_split.ipynb) and Merlin store creation
    notebooks
  repo: https://github.com/theislab/scTab
- name: NVIDIA-Merlin dataloader
  role: Reads the donor-split, normalized corpus from the on-disk Merlin store into
    GPU batches for TabNet training
  repo: https://github.com/NVIDIA-Merlin/dataloader
- name: NumPy
  role: Implements size-factor normalization (sf_normalize) applied to cells after
    the donor split
- name: cellxgene-census
  role: Queries and subsets the CELLxGENE census with donor and assay metadata used
    to define the split
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-024-51059-5
    title: 'scTab: Scaling cross-tissue single-cell annotation models'
  source_content_hash: sha256:6f4fe35edafb7dcbc967de9e84ede4d7646f67c89174e2ed47bb5123e869eadd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/theislab/scTab
  tools:
  - NVIDIA-Merlin dataloader
  - NumPy
  - TabNet
  - cellxgene-census
  - scTab
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# single-cell-donor-based-data-splitting

## Summary

Splits a large single-cell RNA-seq corpus into train/val/test partitions by donor (batch) rather than by cell, so that classifier performance measures generalization to unseen donors. This is essential for honestly quantifying whether scaling gains come from donor diversity versus cell count.

## When to use

Use when training or evaluating cross-tissue cell type classifiers on multi-donor scRNA-seq corpora where test cells must come from donors never seen during training.

## When NOT to use

- When the research question is within-donor performance (e.g., annotating new cells from the same donors), a cell-level random split is appropriate and a donor split would underestimate attainable accuracy.
- When the corpus lacks reliable donor/batch metadata or cell types are observed in fewer than 30 donors, held-out-donor generalization cannot be reliably quantified.
- When data is not primary (is_primary_data == False), donor-based splitting cannot prevent train/test label leakage from duplicated observations.

## Inputs

- CELLxGENE census Cell-by-Gene (CxG) corpus (version 2023-05-15) with donor/batch metadata and cell type annotations
- Cell Ontology child-node lookup matrix for subtype expansion
- Filter criteria: is_primary_data == True, assay in PROTOCOLS, >=5000 cells per cell type, >=30 donors per cell type, >=7 ontology parent nodes, 19,331 protein-coding genes

## Outputs

- Donor-disjoint train (0.7), validation (0.15), and test (0.15) cell partitions with fixed held-out donor sets
- Donor-based training subsamples at 15%, 30%, 50%, 70%, 100% of training donors (plus cell-based subsamples for comparison)
- On-disk Merlin-format data store of size-factor-normalized (10,000 counts) and log1p-transformed cells

## How to apply

From the CELLxGENE census corpus (here: version 2023-05-15, filtered to primary data, supported assays, 'native cell' subtypes, >=5000 cells per type, >=30 donors per type, >=7 ontology parent nodes, 19,331 protein-coding genes), group cells by donor and partition donors — not cells — into 70% train, 15% val, and 15% test, keeping val/test donor sets fixed across all experiments. After splitting, normalize each cell to 10,000 counts, log1p-transform, and write the store to the on-disk Merlin format for GPU dataloading. For scaling experiments, create donor-based training subsamples at 15%, 30%, 50%, 70%, and 100% of training donors, and optionally contrast them with cell-based subsamples that ignore donor identity. Rationale: donor-based splits prevent label leakage from shared batch effects, so macro F1 and cross-entropy measured on the held-out donor test set reflect true cross-donor generalization.

## Related tools

- **scTab** (Training pipeline and repository containing the donor-based train/val/test split notebook (02_create_train_val_test_split.ipynb) and Merlin store creation notebooks) — https://github.com/theislab/scTab
- **NVIDIA-Merlin dataloader** (Reads the donor-split, normalized corpus from the on-disk Merlin store into GPU batches for TabNet training) — https://github.com/NVIDIA-Merlin/dataloader
- **NumPy** (Implements size-factor normalization (sf_normalize) applied to cells after the donor split)
- **cellxgene-census** (Queries and subsets the CELLxGENE census with donor and assay metadata used to define the split)

## Examples

```
python notebooks/store_creation/02_create_train_val_test_split.ipynb  # donor-based 0.7/0.15/0.15 split of the CELLxGENE census corpus
```

## Evaluation signals

- No donor appears in more than one of the train (0.7), validation (0.15), and test (0.15) partitions — verify donor ID disjointness across splits.
- Each retained cell type is observed in at least 30 donors so held-out-donor performance can be quantified.
- Total corpus matches 22,189,056 cells x 19,331 protein-coding genes; train split is 0.7 of the full dataset (≈15.2M training cells at 100%).
- Val and test donor sets are identical across all subsampling regimes (15%–100%), enabling apples-to-apples scaling curves.
- Under the donor-based regime, macro F1 improves markedly with training size (minimum 2.1M→15.2M difference of 0.0454 ± 0.0083; median 0.1219 ± 0.0212), whereas cell-based subsampling yields only weak scaling — a divergence that confirms the split is truly donor-driven.

## Limitations

- Donor-based splits require cells to be primary data; duplicated or non-primary observations leak labels between train and test.
- Plain macro F1 on donor-held-out test sets may not reflect the complexity of datasets imbalanced across cell types, tissues, and sequencing protocols.
- Cell Ontology incompleteness and inconsistent annotation granularity in CELLxGENE affect classification metrics regardless of how donors are split.

## Evidence

- [methods] Split data into train, val and test sets (donor-based split): "2. `02_create_train_val_test_split.ipynb`: Split data into train, val and test sets"
- [methods] Each cell type observed in at least 30 donors: "Each cell type has to be observed in at least `30` donors to reliably quantify whether the classifier can generalize to new donors."
- [methods] Primary data filter prevents leakage: "Data has to be primary data to prevent label leakage between the train and test set: `is_primary_data == True`"
- [results] Scaling driven by batch diversity: [evidence span withheld — anchor status: not_found]
- [methods] Normalization pipeline before store creation: "1. Normalize each cell to have `10000` counts (size factor normalization)"
- [readme] Minimal data subset available for pipeline reuse: "Minimal subset of the training, validation and test data: https://pklab.med.harvard.edu/felix/data/merlin_cxg_2023_05_15_sf-log1p_minimal.tar.gz (0.5GB)"
