---
name: multi-dataset-deg-union-feature-selection
description: Use when building a supervised malignant-vs-normal classifier from multiple
  scRNA-seq training datasets and you need a robust, pan-cancer feature gene list
  that tolerates inter-tumor heterogeneity.
when_to_use_negative:
- Your model features are already defined (e.g., you have the released ordered_feature.tsv
  or combined_tumor_up_down_degs.txt) — no need to recompute the union
- You need genes consistently DE across ALL datasets (intersection) rather than a
  heterogeneous pan-cancer feature set — the union deliberately retains dataset-specific
  DEGs
- Cell labels have not been calibrated against curated cancer gene signatures — this
  DEG step operates on the signature-calibrated training set, not raw original annotations
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: scMalignantFinder
  role: Framework whose logistic regression classifier consumes the 2707-gene union
    feature set; feature list and training data deposited at Zenodo record 17888140
  repo: https://github.com/Jonyyqn/scMalignantFinder
- name: Seurat
  role: Runs FindAllMarkers per training dataset to compute malignant-vs-normal DEGs
    with |log2FC|>=0.5, P<0.01, pct.1>=0.2 thresholds
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s42003-025-07942-y
    title: scMalignantFinder distinguishes malignant cells in single-cell and spatial
      transcriptomics by leveraging cancer signatures
  source_content_hash: sha256:00d02ee67845694318982e729ff03dec35188327248f5f0eaa4bd1d40390afd3
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/Jonyyqn/scMalignantFinder
  repo_url: https://github.com/Jonyyqn/scMalignantFinder
  tools:
  - Seurat
  - scMalignantFinder
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# multi-dataset-deg-union-feature-selection

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Selects model features for malignant-cell classification by running per-dataset differential expression (malignant vs normal epithelial cells) in Seurat and taking the UNION (not intersection) of DEG sets across datasets, capturing inter-tumor heterogeneity that intersection-based selection overlooks.

## When to use

Use when building a supervised malignant-vs-normal classifier from multiple scRNA-seq training datasets and you need a robust, pan-cancer feature gene list that tolerates inter-tumor heterogeneity.

## When NOT to use

- Your model features are already defined (e.g., you have the released ordered_feature.tsv or combined_tumor_up_down_degs.txt) — no need to recompute the union
- You need genes consistently DE across ALL datasets (intersection) rather than a heterogeneous pan-cancer feature set — the union deliberately retains dataset-specific DEGs
- Cell labels have not been calibrated against curated cancer gene signatures — this DEG step operates on the signature-calibrated training set, not raw original annotations

## Inputs

- Signature-calibrated training set combine_training.h5ad with cell labels in adata.obs['Raw_annotation'] ('Malignant'/'Tumor' vs 'Normal')
- Per-dataset malignant and normal epithelial cell annotations (training datasets analyzed separately)

## Outputs

- combined_tumor_up_down_degs.txt: tab-delimited list of 2707 gene symbols (1656 upregulated + 1051 downregulated in malignant cells)
- Input expression matrix restricted to the 2707 DEG features for the logistic regression classifier

## How to apply

Load the signature-calibrated training set (combine_training.h5ad) with cell labels from adata.obs['Raw_annotation'], treating 'Malignant'/'Tumor' as malignant and 'Normal' as the comparator. For EACH training dataset separately, run Seurat FindAllMarkers comparing malignant vs normal epithelial cells with thresholds |log2FC|>=0.5, P<0.01, and pct.1>=0.2, then split each result into up- and downregulated sets by the sign of log2 fold change. Take the union of the upregulated sets across datasets (yielding 1656 genes) and the union of the downregulated sets (1051 genes), concatenate them into a single 2707-gene feature matrix, and save as the tab-delimited combined_tumor_up_down_degs.txt. The union strategy is deliberate: intersection-based selection of genes consistently DE across datasets can overlook substantial inter-tumor heterogeneity.

## Related tools

- **scMalignantFinder** (Framework whose logistic regression classifier consumes the 2707-gene union feature set; feature list and training data deposited at Zenodo record 17888140) — https://github.com/Jonyyqn/scMalignantFinder
- **Seurat** (Runs FindAllMarkers per training dataset to compute malignant-vs-normal DEGs with |log2FC|>=0.5, P<0.01, pct.1>=0.2 thresholds)

## Examples

```
R: markers <- FindAllMarkers(seurat_obj, only.pos=FALSE, logfc.threshold=0.5, min.pct=0.2); up <- union_up(markers[markers$avg_log2FC>0 & markers$p_val_adj<0.01,])
```

## Evaluation signals

- combined_tumor_up_down_degs.txt exists and contains exactly 2707 gene symbols
- Gene counts split correctly: 1656 upregulated and 1051 downregulated in malignant cells
- File content matches the reference feature list deposited at Zenodo (zenodo.org/records/17888140, file combined_tumor_up_down_degs.txt)
- Per-dataset DEG runs applied the specified thresholds: |log2FC|>=0.5, P<0.01, pct.1>=0.2
- Union (not intersection) was taken across datasets — every gene appearing as DEG in at least one dataset is present

## Limitations

- Union selection intentionally keeps dataset-specific DEGs; it does not enforce cross-dataset consistency, which differs from approaches selecting only genes consistently DE across datasets
- Features are defined for malignant-vs-normal-epithelial comparisons within cancerous lineages; the resulting classifier assumes prior identification of basic cell types
- DEGs are derived from the calibrated training set only; untested for cancers with unknown or controversial origins

## Evidence

- [results] From the calibrated training set, a set of 2707 DEGs (1656 upregulated and 1051 downregulated in malignant cells) was used to construct the input expression matrix for the logistic regression classifier.: [evidence span withheld — anchor status: not_found]
