---
name: differential-gene-expression-analysis
description: Use when building a malignant-vs-normal epithelial classifier feature
  set from multiple annotated scRNA-seq training datasets, or when you need to combine
  per-dataset DEG lists into one pan-cancer gene panel.
when_to_use_negative:
- A feature list already exists (e.g., ordered_feature.tsv or combined_tumor_up_down_degs.txt
  from Zenodo record 17888140) — no need to re-derive DEGs
- The goal is malignant cell identification from a single already-annotated dataset
  rather than constructing a pan-cancer training feature set
- The comparison is between malignant cells and all non-malignant cells rather than
  their originating normal epithelial counterpart — scMalignantFinder is specifically
  designed for the latter
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0622
tools:
- name: Seurat
  role: R toolkit used to run FindAllMarkers per training dataset with |log2FC|>=0.5,
    P<0.01, pct.1>=0.2 thresholds
- name: scMalignantFinder
  role: Pipeline whose calibrated training set supplies the malignant/normal epithelial
    labels and whose classifier consumes the derived 2707-DEG feature matrix
  repo: github.com/Jonyyqn/scMalignantFinder
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s42003-025-07942-y
    title: scMalignantFinder distinguishes malignant cells in single-cell and spatial
      transcriptomics by leveraging cancer signatures
  source_content_hash: sha256:3f86cc2ec8d8617464fdb656f144b55a00205d0dcd7e992e9ed8e8b10233fc12
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/Jonyyqn/scMalignantFinder
  tools:
  - Seurat
  - scMalignantFinder
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# differential-gene-expression-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Derives the scMalignantFinder model feature set by running per-dataset differential expression (Seurat FindAllMarkers) between malignant and normal epithelial cells in the signature-calibrated training set, then taking the union (not intersection) of upregulated and downregulated DEG sets into a single 2707-gene feature matrix (1656 up + 1051 down).

## When to use

Use when building a malignant-vs-normal epithelial classifier feature set from multiple annotated scRNA-seq training datasets, or when you need to combine per-dataset DEG lists into one pan-cancer gene panel.

## When NOT to use

- A feature list already exists (e.g., ordered_feature.tsv or combined_tumor_up_down_degs.txt from Zenodo record 17888140) — no need to re-derive DEGs
- The goal is malignant cell identification from a single already-annotated dataset rather than constructing a pan-cancer training feature set
- The comparison is between malignant cells and all non-malignant cells rather than their originating normal epithelial counterpart — scMalignantFinder is specifically designed for the latter

## Inputs

- combine_training.h5ad — signature-calibrated training set with cell labels in adata.obs['Raw_annotation'] ('Malignant'/'Tumor' vs 'Normal')
- Per-dataset Seurat objects subset to malignant and normal epithelial cells
- Seurat FindAllMarkers DE results tables per dataset

## Outputs

- combined_tumor_up_down_degs.txt — tab-delimited file with exactly 2707 gene symbols (1656 upregulated + 1051 downregulated)
- Upregulated (1656) and downregulated (1051) gene union sets used as logistic regression model features

## How to apply

Load combine_training.h5ad and read cell labels from adata.obs['Raw_annotation'], treating 'Malignant' and 'Tumor' as malignant and 'Normal' as the non-malignant comparator. For each training dataset separately, run Seurat FindAllMarkers comparing malignant vs normal epithelial cells with thresholds |log2FC| >= 0.5, P < 0.01, and pct.1 >= 0.2, then split results into upregulated and downregulated sets by the sign of log2FC. Take the union (not intersection) of upregulated sets across all datasets to obtain 1656 upregulated genes and the union of downregulated sets to obtain 1051 downregulated genes; the union-based strategy was chosen deliberately because selecting only genes consistently DE across datasets can overlook substantial inter-tumor heterogeneity. Concatenate the two union sets into a single 2707-gene list and save it as the tab-delimited file combined_tumor_up_down_degs.txt, which feeds the logistic regression classifier's input expression matrix.

## Related tools

- **Seurat** (R toolkit used to run FindAllMarkers per training dataset with |log2FC|>=0.5, P<0.01, pct.1>=0.2 thresholds)
- **scMalignantFinder** (Pipeline whose calibrated training set supplies the malignant/normal epithelial labels and whose classifier consumes the derived 2707-DEG feature matrix) — github.com/Jonyyqn/scMalignantFinder

## Examples

```
markers <- FindAllMarkers(seurat_obj, only.pos = FALSE, logfc.threshold = 0.5, min.pct = 0.2); sig <- subset(markers, p_val < 0.01)
```

## Evaluation signals

- combined_tumor_up_down_degs.txt exists and contains exactly 2707 gene symbols (1656 upregulated + 1051 downregulated)
- The gene list matches the reference feature file deposited at zenodo.org/records/17888140 (file combined_tumor_up_down_degs.txt)
- Per-dataset DEG results honor the stated thresholds: |log2FC| >= 0.5, P < 0.01, pct.1 >= 0.2
- Up/down gene sets are unions across datasets, not intersections — count should not drop below per-dataset maxima
- Each gene is assigned to the up or down set consistently with the sign of its log2 fold change

## Limitations

- DEGs are defined only between malignant cells and normal epithelial cells; skill presumes accurate malignancy labels from the signature-calibrated training set (Raw_annotation: 'Malignant'/'Tumor'/'Normal')
- Union-based feature selection intentionally retains genes that are not consistently DE across datasets, so the panel is broader and less specific than a consistently-DE (intersection) gene set
- Downstream classifier is restricted to cancerous lineages and requires prerequisite identification of basic cell types

## Evidence

- [methods] per-dataset DE thresholds: [evidence span withheld — anchor status: not_found]
- [methods] union strategy: [evidence span withheld — anchor status: not_found]
