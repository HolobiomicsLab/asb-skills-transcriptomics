---
name: scrna-seq-malignant-vs-epithelial-deg-testing
description: Use when building malignant-vs-normal cell classifier features from a
  signature-calibrated training set and you need per-dataset DEG testing (Seurat FindAllMarkers)
  followed by a cross-dataset union.
when_to_use_negative:
- A pre-built feature list already exists (e.g. the deposited ordered_feature.tsv
  or combined_tumor_up_down_degs.txt at Zenodo record 17888140) — reuse it instead
  of re-running DEG testing
- Cells have not yet been annotated into basic cell types, so no malignant vs normal
  epithelial comparator exists — scMalignantFinder relies on the prerequisite identification
  of basic cell types within cancerous lineages
- The comparison of interest is malignant vs all other non-malignant cells rather
  than vs originating normal epithelial cells — this skill's calibration and comparator
  are specifically normal epithelial cells
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: scMalignantFinder
  role: Pipeline whose logistic regression classifier consumes the 2707-gene union
    DEG feature set derived by this skill
  repo: github.com/Jonyyqn/scMalignantFinder
- name: Seurat
  role: Runs per-dataset differential expression (FindAllMarkers) comparing malignant
    vs normal epithelial cells with |log2FC|>=0.5, P<0.01, pct.1>=0.2
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s42003-025-07942-y
    title: scMalignantFinder distinguishes malignant cells in single-cell and spatial
      transcriptomics by leveraging cancer signatures
  source_content_hash: sha256:afdb1cec9a353cc3990a22515e94cb3c7f27b1da449318dc4d9110989df5fcaf
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
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# scrna-seq-malignant-vs-epithelial-deg-testing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Derives the scMalignantFinder model feature set by running per-dataset differential expression between malignant and normal epithelial cells and taking the union of up- and downregulated DEGs, yielding a 2707-gene feature list (1656 up, 1051 down).

## When to use

Use when building malignant-vs-normal cell classifier features from a signature-calibrated training set and you need per-dataset DEG testing (Seurat FindAllMarkers) followed by a cross-dataset union.

## When NOT to use

- A pre-built feature list already exists (e.g. the deposited ordered_feature.tsv or combined_tumor_up_down_degs.txt at Zenodo record 17888140) — reuse it instead of re-running DEG testing
- Cells have not yet been annotated into basic cell types, so no malignant vs normal epithelial comparator exists — scMalignantFinder relies on the prerequisite identification of basic cell types within cancerous lineages
- The comparison of interest is malignant vs all other non-malignant cells rather than vs originating normal epithelial cells — this skill's calibration and comparator are specifically normal epithelial cells

## Inputs

- combine_training.h5ad — signature-calibrated training set (Zenodo record 17888140) with cell labels in adata.obs['Raw_annotation'] ('Malignant'/'Tumor' = malignant; 'Normal' = comparator)
- Per-dataset Seurat objects for the individual training datasets

## Outputs

- combined_tumor_up_down_degs.txt — tab-delimited list of exactly 2707 gene symbols (1656 upregulated + 1051 downregulated) used as model features

## How to apply

Load the calibrated training set combine_training.h5ad and read cell labels from adata.obs['Raw_annotation'], treating both 'Malignant' and 'Tumor' as malignant and 'Normal' as the non-malignant comparator. For each training dataset separately, run Seurat FindAllMarkers comparing malignant vs normal epithelial cells with thresholds |log2FC|>=0.5, P<0.01, and pct.1>=0.2. Split each per-dataset result into upregulated and downregulated gene sets by the sign of the log2 fold change. Take the union (not the intersection) of the upregulated sets across datasets (1656 genes) and of the downregulated sets (1051 genes); the union is deliberately chosen because intersection-based feature selection can overlook substantial inter-tumor heterogeneity. Concatenate the two union sets into a single 2707-gene tab-delimited feature file, combined_tumor_up_down_degs.txt, which becomes the input expression-matrix feature set for the logistic regression classifier.

## Related tools

- **scMalignantFinder** (Pipeline whose logistic regression classifier consumes the 2707-gene union DEG feature set derived by this skill) — github.com/Jonyyqn/scMalignantFinder
- **Seurat** (Runs per-dataset differential expression (FindAllMarkers) comparing malignant vs normal epithelial cells with |log2FC|>=0.5, P<0.01, pct.1>=0.2)

## Examples

```
markers <- FindAllMarkers(seurat_obj, logfc.threshold = 0.5, min.pct = 0.2); degs <- subset(markers, p_val < 0.01)
```

## Evaluation signals

- combined_tumor_up_down_degs.txt exists and contains exactly 2707 gene symbols (1656 upregulated + 1051 downregulated)
- The produced feature list matches the reference file combined_tumor_up_down_degs.txt deposited at zenodo.org/records/17888140
- Each per-dataset DEG call respected the thresholds |log2FC|>=0.5, P<0.01, and pct.1>=0.2
- Up/down gene sets were formed by union across datasets, not intersection; sign split is consistent with the log2 fold-change direction
- Every gene in the final 2707-gene set is present in the feature matrix used by the logistic regression classifier

## Limitations

- Feature selection is restricted to DEGs between malignant cells and normal epithelial cells within cancerous lineages; the classifier relies on the prerequisite identification of basic cell types
- Capabilities are untested for cancers with unknown or controversial origins (e.g. synovial sarcoma and certain brain tumors)
- The calibrated malignant set itself excluded 2.5% of training cells that lost up- or downregulated cancer signatures during calibration, so DEG input depends on the upstream signature calibration step

## Evidence
