---
name: cluster-based-malignant-cell-refinement
description: Use when scATOMIC has annotated cells as cancer at the terminal layer
  but normal cells from the cancer's tissue of origin may be present in the tumour
  biopsy and must be distinguished from true malignant cells.
when_to_use_negative:
- Running on normal (non-cancer) tissue samples — the default scATOMIC model assumes
  at least some cancer cells are present in the biopsy; use the normal_tissue parameter
  instead
- Datasets pooling cells across multiple patients/biopsies — inter-patient cancer
  cell-specific effects interfere with the cancer signature scoring module and classification
  confidence
- Scoring cells that were not annotated as cancer at the terminal layer — the up-/down-regulated
  program scoring applies only to cells that received an original cancer-type annotation
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: scATOMIC
  role: Provides run_scATOMIC() hierarchical cancer-cell annotations and create_summary_matrix()
    for consolidating per-layer predictions into the Cancer vs Normal classification_confidence
  repo: https://github.com/abelson-lab/scATOMIC
- name: Seurat
  role: AddModuleScore is used to score the OncoDB-derived cancer-type up-regulated
    and down-regulated gene programs in terminal-layer cancer cells
- name: cutoff.scATOMIC
  role: Forked cutoff package (finite mixture model/EM) used by scATOMIC to derive
    classification cutoffs separating bimodal score distributions
  repo: https://github.com/inofechm/cutoff.scATOMIC
- name: Rmagic
  role: Performs MAGIC imputation of the count matrix before run_scATOMIC() classification,
    as enabled in the workflow
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-023-37353-8
    title: Pan-cancer classification of single cells in the tumour microenvironment
  source_content_hash: sha256:bb48c1c2c1ed815f90a7cc0d6ad53de7764a5c9749af65338acb04a5af4c38f5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/abelson-lab/scATOMIC
  tools:
  - R
  - Rmagic
  - Seurat
  - cutoff.scATOMIC
  - reticulate
  - scATOMIC
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# cluster-based-malignant-cell-refinement

## Summary

A scATOMIC post-processing module that separates malignant cells from non-malignant tissue-specific cells sharing the same tissue of origin, by scoring cancer-type-specific transcriptional programs and iteratively refining clusters until normal-cell contamination is minimal.

## When to use

Use when scATOMIC has annotated cells as cancer at the terminal layer but normal cells from the cancer's tissue of origin may be present in the tumour biopsy and must be distinguished from true malignant cells.

## When NOT to use

- Running on normal (non-cancer) tissue samples — the default scATOMIC model assumes at least some cancer cells are present in the biopsy; use the normal_tissue parameter instead
- Datasets pooling cells across multiple patients/biopsies — inter-patient cancer cell-specific effects interfere with the cancer signature scoring module and classification confidence
- Scoring cells that were not annotated as cancer at the terminal layer — the up-/down-regulated program scoring applies only to cells that received an original cancer-type annotation

## Inputs

- Sparse gene-by-cell count matrix (e.g. demo_lung_data), filtered to remove cells with >25% mitochondrial reads or <500 unique expressed features
- run_scATOMIC() hierarchical prediction table (per-cell annotations at each layer, terminal-layer cancer calls)
- OncoDB-derived cancer-type up-regulated and down-regulated gene program lists (pan_cancer_upreg / pan_cancer_downreg, |log2FC|>1, adj P<0.01)

## Outputs

- pan_cancer_cluster label per cell (cluster partition with ≤20% normal cells per cluster)
- Per-cell Cancer vs Normal classification_confidence label from create_summary_matrix()
- Table of one row per filtered cell with layer predictions, pan_cancer_cluster, and classification_confidence (e.g. cancer_vs_normal_labels.csv)

## How to apply

Run run_scATOMIC() on the filtered sparse gene-by-cell count matrix (cells with >25% mitochondrial reads or <500 unique expressed features removed) with Rmagic/MAGIC imputation, then restrict to cells called cancer at the terminal layer. Score the OncoDB-derived pan_cancer_upreg / pan_cancer_downreg gene programs (|log2FC|>1, adj P<0.01) in those cells with Seurat's AddModuleScore, capturing cancer-type-specific up- and down-regulated programs. Cluster the signature-scored cells and iteratively drop/refine clusters whose fraction of normal cells exceeds 20%, repeating until every retained cluster has ≤20% normal cells; label the final partition pan_cancer_cluster. Consolidate per-layer predictions with create_summary_matrix() (modify_results = TRUE assuming a single cancer type, min_prop = 0.5) to derive the per-cell Cancer vs Normal classification_confidence. Because inter-patient cancer cell-specific effects interfere with the signature scoring module, apply this only to individual patient/biopsy samples.

## Related tools

- **scATOMIC** (Provides run_scATOMIC() hierarchical cancer-cell annotations and create_summary_matrix() for consolidating per-layer predictions into the Cancer vs Normal classification_confidence) — https://github.com/abelson-lab/scATOMIC
- **Seurat** (AddModuleScore is used to score the OncoDB-derived cancer-type up-regulated and down-regulated gene programs in terminal-layer cancer cells)
- **cutoff.scATOMIC** (Forked cutoff package (finite mixture model/EM) used by scATOMIC to derive classification cutoffs separating bimodal score distributions) — https://github.com/inofechm/cutoff.scATOMIC
- **Rmagic** (Performs MAGIC imputation of the count matrix before run_scATOMIC() classification, as enabled in the workflow)

## Examples

```
summary_mat <- create_summary_matrix(predictions = scATOMIC_predictions, modify_results = TRUE, min_prop = 0.5)
```

## Evaluation signals

- Output CSV exists with exactly one row per filtered input cell and contains both pan_cancer_cluster and classification_confidence fields
- Every retained pan_cancer_cluster has ≤20% normal cells after iterative refinement (cluster count only decreases across refinement rounds)
- On the lung demo dataset, the scATOMIC_pred table yields a majority of cells labeled confident Lung Cancer Cell, matching the tutorial's reported class distribution
- create_summary_matrix() was called with modify_results = TRUE (single cancer type) and min_prop = 0.5, and classification_confidence is populated for all cells
- Module scores were computed only on cells with a terminal-layer cancer annotation, using the |log2FC|>1, adj P<0.01 program definitions

## Limitations

- Inter-patient cancer cell-specific effects interfere with scATOMIC's cancer signature scoring module and confidence in classifications, so the module should be run only on individual patient/biopsy samples
- The default scATOMIC model assumes the presence of at least some cancer cells in a biopsy and should not be used on normal tissue samples
- A degree of discordancy exists between scATOMIC and CNV inference; using scATOMIC in conjunction with CNV inference to annotate cancer cells is recommended

## Evidence

- [readme] Inter-patient cancer cell specific effects can interfere with scATOMICs cancer signature scoring module and confidence in classifications.: "Inter-patient cancer cell specific effects can interfere with scATOMICs cancer signature scoring module and confidence in classifications."
- [methods] After running scATOMIC we generate a summary of the intermediate results with create_summary_matrix().: "After running scATOMIC we generate a summary of the intermediate results with create_summary_matrix()."
- [discussion] using scATOMIC in conjunction with CNV inference to annotate cancer cells and their type is recommended: "using scATOMIC in conjunction with CNV inference to annotate cancer cells and their type is recommended"
- [readme] We strongly recommend running scATOMIC only on individual patient/biopsy samples.: "We strongly recommend running scATOMIC only on individual patient/biopsy samples."
- [intro] Solely relying on the presence of inferred CNVs to annotate malignant cells may lead to false negatives or undefined cells in cancers with minimal genomic structural variation: "Solely relying on the presence of inferred CNVs to annotate malignant cells may lead to false negatives or undefined cells in cancers with minimal genomic structural variation"
