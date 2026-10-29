---
name: cross-platform-cell-type-annotation
description: Use when a snRNA-seq/scRNA-seq query dataset from one sequencing platform
  must be labeled using a reference built from other platforms, or when benchmarking
  annotation tools on externally harmonized datasets such as the Allen Brain Atlas.
when_to_use_negative:
- Query data is from a disease context whose expression profiles deviate strongly
  from the normal reference — use SELINA's disease mode models or user-supplied disease
  references instead
- The species or cell types of interest are outside the human reference atlas coverage
  (transfer to other species is only anticipated, not demonstrated)
- A comprehensive, uniformly annotated reference atlas already exists for the exact
  tissue and the goal is not cross-platform transfer or tool benchmarking
edam_operation: http://edamontology.org/operation_0313
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: SELINA
  role: 'Primary annotation framework: preprocess, train, and predict cell types across
    platforms via MADA and autoencoder fine-tuning'
  repo: https://github.com/SELINA-team/SELINA.py
- name: Garnett
  role: Marker-based comparator tool benchmarked on the same filtered datasets
- name: SCINA
  role: Marker-based comparator tool benchmarked on the same filtered datasets
- name: scmap
  role: Reference-based comparator tool benchmarked on the same filtered datasets
- name: scPred
  role: Reference-based comparator tool benchmarked on the same filtered datasets
- name: SingleR
  role: Reference-based comparator tool benchmarked on the same filtered datasets
- name: CHETAH
  role: Reference-based comparator tool benchmarked on the same filtered datasets
- name: SingleCellNet
  role: Reference-based comparator tool benchmarked on the same filtered datasets
- name: Python
  role: SELINA Python package environment (conda) used to run preprocess/train/predict
  repo: https://github.com/SELINA-team/SELINA.py
- name: R
  role: SELINA R package runtime and environment for Seurat-dependent preprocessing
    steps
  repo: https://github.com/SELINA-team/SELINA.py
- name: presto
  role: Fast differential expression used in the SELINA preprocessing/annotation pipeline
  repo: https://github.com/immunogenomics/presto
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.crmeth.2023.100577
    title: Single-cell assignment using multiple-adversarial domain adaptation network
      with large-scale references
  source_content_hash: sha256:b842b14ae2b8c7127d6b10849001341b34338e63c04c8d6605164442c7409006
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/SELINA-team/SELINA.py
  repo_url: https://github.com/SELINA-team/SELINA.py
  tools:
  - CHETAH
  - Garnett
  - Python
  - R
  - SCINA
  - SELINA
  - SingleCellNet
  - SingleR
  - presto
  - scPred
  - scmap
  edam_operation: http://edamontology.org/operation_0313
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# cross-platform-cell-type-annotation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Annotate single-cell RNA-seq query data generated on different platforms (e.g., 10x vs. Smart-seq) by transferring labels from a large harmonized reference atlas using SELINA's multi-adversarial domain adaptation workflow, and benchmark against reference-based and marker-based annotation tools.

## When to use

Use when a snRNA-seq/scRNA-seq query dataset from one sequencing platform must be labeled using a reference built from other platforms, or when benchmarking annotation tools on externally harmonized datasets such as the Allen Brain Atlas.

## When NOT to use

- Query data is from a disease context whose expression profiles deviate strongly from the normal reference — use SELINA's disease mode models or user-supplied disease references instead
- The species or cell types of interest are outside the human reference atlas coverage (transfer to other species is only anticipated, not demonstrated)
- A comprehensive, uniformly annotated reference atlas already exists for the exact tissue and the goal is not cross-platform transfer or tool benchmarking

## Inputs

- snRNA-seq query count matrices (plain/mtx/h5 formats), e.g., Allen Brain Atlas 10x (76,533 nuclei) and Smart-seq (49,495 nuclei) datasets
- Pre-curated reference atlas data (1.7 million cells, 230 human cell types) or pre-trained models from SELINA-reference
- Reference-annotated cell type labels for the >1,000-cell-per-type subsets

## Outputs

- pre-trained_params.pt (trained model parameters) and pre-trained_meta.pkl (cell types and genes)
- query_predictions.txt: predicted cell type per query cell
- query_probability.txt: per-cell probability over reference cell types
- Per-tool accuracy and MacroF1 across the 19 cell types, averaged over datasets (e.g., SELINA 93.32% accuracy, MacroF1 0.9336)

## How to apply

Download the query datasets (e.g., the 76,533-nucleus 10x and 49,495-nucleus Smart-seq Allen Brain Atlas sets), restrict to cell types with >1,000 cells, and run `selina preprocess` to normalize profiles, convert genes to hg38 symbol names, and compute PCA (30 PCs), UMAP, and Louvain clustering (resolution 0.6) with default QC cutoffs (count 1000, gene 500, cell 10). Train the model with `selina train` on preprocessed reference data to produce pre-trained_params.pt and pre-trained_meta.pkl, then run `selina predict` with --prob-cutoff 0.9 to get per-cell predictions and probabilities. For benchmarking, run comparators (Garnett, SCINA, scmap, scPred, SingleR, CHETAH, SingleCellNet, plus the remaining comparator) with their reference-based or marker-based protocols on the same filtered data, compute accuracy and MacroF1 per dataset, average across datasets, and rank the tools. The domain-adaptation design (SMOTE balancing, MADA pre-training, autoencoder fine-tuning) is what makes cross-platform label transfer robust to batch effects.

## Related tools

- **SELINA** (Primary annotation framework: preprocess, train, and predict cell types across platforms via MADA and autoencoder fine-tuning) — https://github.com/SELINA-team/SELINA.py
- **Garnett** (Marker-based comparator tool benchmarked on the same filtered datasets)
- **SCINA** (Marker-based comparator tool benchmarked on the same filtered datasets)
- **scmap** (Reference-based comparator tool benchmarked on the same filtered datasets)
- **scPred** (Reference-based comparator tool benchmarked on the same filtered datasets)
- **SingleR** (Reference-based comparator tool benchmarked on the same filtered datasets)
- **CHETAH** (Reference-based comparator tool benchmarked on the same filtered datasets)
- **SingleCellNet** (Reference-based comparator tool benchmarked on the same filtered datasets)
- **Python** (SELINA Python package environment (conda) used to run preprocess/train/predict) — https://github.com/SELINA-team/SELINA.py
- **R** (SELINA R package runtime and environment for Seurat-dependent preprocessing steps) — https://github.com/SELINA-team/SELINA.py
- **presto** (Fast differential expression used in the SELINA preprocessing/annotation pipeline) — https://github.com/immunogenomics/presto

## Examples

```
selina predict --query-expr ./res/query_expr.txt --model ./res/pre-trained_params.pt --seurat ./res/query_res.rds --path-out ./res/predict/ --prob-cutoff 0.9
```

## Evaluation signals

- The benchmark table reproduces Figure 5B ordering: SELINA ranks second of 9 tools with average accuracy within tolerance of 93.32% and MacroF1 within tolerance of 0.9336
- `selina train` produces both pre-trained_params.pt and pre-trained_meta.pkl before prediction can proceed
- `selina preprocess` yields four outputs: query_res.rds (Seurat object), query_expr.txt, query_cluster.png, and query_cluster_DiffGenes.tsv
- `selina predict` with --prob-cutoff 0.9 yields query_predictions.txt and query_probability.txt with one prediction/probability row per query cell
- Both filtered Allen datasets (10x and Smart-seq) contain only the 19 cell types with >1,000 cells before annotation

## Limitations

- The reference atlas remains imbalanced in tissue-specific datasets and cell-type abundance despite its large scale
- Disease expression profiles are too diverse to cover comprehensively; the pre-trained disease models only cover non-small-cell lung carcinoma, type 2 diabetes, and Alzheimer's disease, and fine-tuning is skipped (--disease flag) for disease queries
- No comprehensive reference atlas with unified cell types exists yet; SELINA reference updates and transfer to other species remain future plans, and graph-based algorithms for structural information are planned but not implemented

## Evidence
