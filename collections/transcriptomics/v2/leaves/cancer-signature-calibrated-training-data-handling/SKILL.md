---
name: cancer-signature-calibrated-training-data-handling
description: 'Use when building a malignant-cell classifier''s feature matrix from
  a multi-dataset calibrated training set: you have malignant/normal-annotated cells
  (e.g. adata.obs[''Raw_annotation''] with ''Malignant''/''Tumor'' vs ''Normal'')
  and need union-based DEG feature selection rather than intersection.'
when_to_use_negative:
- You already have the deposited feature list (ordered_feature.tsv or combined_tumor_up_down_degs.txt
  from Zenodo record 17888140) and only need to run inference — reuse the pretrained
  model.joblib instead of re-deriving DEGs.
- Your goal requires genes consistently differentially expressed across ALL datasets
  (intersection), which contradicts the union strategy and would lose inter-tumor
  heterogeneity.
- Your data contains cells of unknown/controversial origin (e.g. synovial sarcoma,
  certain brain tumors) or is not a cancerous epithelial lineage — scMalignantFinder
  relies on prerequisite identification of basic cell types.
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: Seurat
  role: Runs per-dataset differential expression via FindAllMarkers comparing malignant
    vs normal epithelial cells with |log2FC| >= 0.5, P < 0.01, pct.1 >= 0.2
- name: scMalignantFinder
  role: Provides the calibrated training set (combine_training.h5ad) and consumes
    the resulting 2707-gene feature matrix for its logistic regression classifier
  repo: github.com/Jonyyqn/scMalignantFinder
- name: Scanpy
  role: Used for total-count normalization of input expression data in the surrounding
    pipeline (sc.pp.normalize_total, target_sum=1e4)
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s42003-025-07942-y
    title: scMalignantFinder distinguishes malignant cells in single-cell and spatial
      transcriptomics by leveraging cancer signatures
  source_content_hash: sha256:195006e18d6d159fe5a81cf08bb075c43b6135d442bf6bb8f9f35e052ff97299
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/Jonyyqn/scMalignantFinder
  tools:
  - Scanpy
  - Seurat
  - scMalignantFinder
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# cancer-signature-calibrated-training-data-handling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Derives the 2707-gene (1656 upregulated + 1051 downregulated) malignant-vs-normal epithelial feature set by taking the union of per-dataset DEGs from the cancer-signature-calibrated training set (combine_training.h5ad), as done in scMalignantFinder. The union strategy intentionally retains inter-tumor heterogeneous markers that intersection-based feature selection would discard.

## When to use

Use when building a malignant-cell classifier's feature matrix from a multi-dataset calibrated training set: you have malignant/normal-annotated cells (e.g. adata.obs['Raw_annotation'] with 'Malignant'/'Tumor' vs 'Normal') and need union-based DEG feature selection rather than intersection.

## When NOT to use

- You already have the deposited feature list (ordered_feature.tsv or combined_tumor_up_down_degs.txt from Zenodo record 17888140) and only need to run inference — reuse the pretrained model.joblib instead of re-deriving DEGs.
- Your goal requires genes consistently differentially expressed across ALL datasets (intersection), which contradicts the union strategy and would lose inter-tumor heterogeneity.
- Your data contains cells of unknown/controversial origin (e.g. synovial sarcoma, certain brain tumors) or is not a cancerous epithelial lineage — scMalignantFinder relies on prerequisite identification of basic cell types.

## Inputs

- Cancer-signature-calibrated training set combine_training.h5ad with cell labels in adata.obs['Raw_annotation']
- Per-dataset malignant and normal epithelial cell annotations ('Malignant'/'Tumor' vs 'Normal')

## Outputs

- Tab-delimited feature list combined_tumor_up_down_degs.txt with exactly 2707 gene symbols (1656 upregulated + 1051 downregulated)
- Ordered/feature gene list usable to construct the input expression matrix for a logistic regression classifier

## How to apply

Load the calibrated training set combine_training.h5ad and read cell labels from adata.obs['Raw_annotation'], treating both 'Malignant' and 'Tumor' as malignant and 'Normal' as the non-malignant epithelial comparator. For each training dataset separately, run Seurat FindAllMarkers comparing malignant vs normal epithelial cells with thresholds |log2FC| >= 0.5, P < 0.01, and pct.1 >= 0.2. Split each per-dataset result into upregulated and downregulated sets by the sign of log2FC, then take the union (NOT the intersection) of upregulated sets across all datasets and the union of downregulated sets. Concatenate the two unions into a single 2707-gene list and save as a tab-delimited file combined_tumor_up_down_degs.txt. The rationale is that intersection-based selection 'potentially overlook[s] substantial inter-tumor heterogeneity', whereas the union preserves cancer-signature patterns across diverse cancer types.

## Related tools

- **Seurat** (Runs per-dataset differential expression via FindAllMarkers comparing malignant vs normal epithelial cells with |log2FC| >= 0.5, P < 0.01, pct.1 >= 0.2)
- **scMalignantFinder** (Provides the calibrated training set (combine_training.h5ad) and consumes the resulting 2707-gene feature matrix for its logistic regression classifier) — github.com/Jonyyqn/scMalignantFinder
- **Scanpy** (Used for total-count normalization of input expression data in the surrounding pipeline (sc.pp.normalize_total, target_sum=1e4))

## Examples

```
import scanpy as sc; adata = sc.read_h5ad('combine_training.h5ad'); labels = adata.obs['Raw_annotation']  # 'Malignant'/'Tumor' vs 'Normal'
```

## Evaluation signals

- combined_tumor_up_down_degs.txt exists and contains exactly 2707 gene symbols
- Gene counts split as 1656 upregulated + 1051 downregulated, matching the reported numbers
- The output file matches the reference feature list deposited at Zenodo (zenodo.org/records/17888140, file combined_tumor_up_down_degs.txt)
- Up/down assignment follows the sign of the per-dataset log2FC, and each class is a union across datasets (size >= any single dataset's DEG count)

## Limitations

- Feature quality depends on the cancer-signature calibration of initially annotated malignant cells; the article notes existing supervised methods are compromised by poorly annotated reference datasets.
- scMalignantFinder is designed to distinguish malignant cells from their originating normal cells (cancerous lineages) only, relying on prerequisite identification of basic cell types.
- Capabilities are untested for cancers with unknown or controversial origins, such as synovial sarcoma and certain brain tumors.

## Evidence
