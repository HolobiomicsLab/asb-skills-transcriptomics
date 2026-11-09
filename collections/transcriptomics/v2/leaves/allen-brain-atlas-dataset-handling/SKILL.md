---
name: allen-brain-atlas-dataset-handling
description: Use when validating or benchmarking a cell-type annotation method on
  external harmonized snRNA-seq query datasets such as the Allen Brain Atlas 10x (76,533
  nuclei) and Smart-seq (49,495 nuclei) sets.
when_to_use_negative:
- Query data are disease snRNA-seq profiles — the paper notes disease expression deviates
  substantially from normal data and recommends users train on their own disease data
  rather than the pre-trained atlas models
- The goal is annotation of non-human species — transfer of SELINA to other species
  is anticipated but not demonstrated in the paper
- Cell types of interest have fewer than 1,000 cells in the query dataset, since the
  benchmark restricts to types with >1,000 cells
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: SELINA
  role: Preprocess query data (normalize, hg38 symbol conversion, PCA/UMAP/Louvain)
    and predict cell types with pre-trained models
  repo: https://github.com/SELINA-team/SELINA.py
- name: Python
  role: Environment for running the selina CLI pipeline (preprocess/train/predict)
  repo: https://github.com/SELINA-team/SELINA.py
- name: R
  role: Runtime for comparator annotation tools and Seurat-based preprocessing outputs
    (query_res.rds)
  repo: https://github.com/SELINA-team/SELINA.py
- name: Garnett
  role: Marker-based comparator annotation tool benchmarked on the same filtered Allen
    datasets
- name: SCINA
  role: Marker-based comparator annotation tool benchmarked on the same filtered Allen
    datasets
- name: scmap
  role: Reference-based comparator annotation tool benchmarked on the same filtered
    Allen datasets
- name: scPred
  role: Reference-based comparator annotation tool benchmarked on the same filtered
    Allen datasets
- name: SingleR
  role: Reference-based comparator annotation tool benchmarked on the same filtered
    Allen datasets
- name: CHETAH
  role: Reference-based comparator annotation tool benchmarked on the same filtered
    Allen datasets
- name: SingleCellNet
  role: Reference-based comparator annotation tool benchmarked on the same filtered
    Allen datasets
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.crmeth.2023.100577
    title: Single-cell assignment using multiple-adversarial domain adaptation network
      with large-scale references
  source_content_hash: sha256:85594f1e7093ae6cb0de4f82f9be373861c80595ba89484a6a98132a7e321388
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
  - scPred
  - scmap
  edam_operation: http://edamontology.org/operation_3803
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5333333333333333
n_claims: 15
---

# allen-brain-atlas-dataset-handling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Workflow for benchmarking cell-type annotation on external harmonized snRNA-seq data from the Allen Brain Atlas: filter to cell types with >1,000 cells, preprocess with SELINA, predict with pre-trained models, and score accuracy/MacroF1 against comparator tools. Used to validate SELINA on data outside its training atlas (93.32% accuracy, MacroF1 0.9336, rank 2 of 9 tools).

## When to use

Use when validating or benchmarking a cell-type annotation method on external harmonized snRNA-seq query datasets such as the Allen Brain Atlas 10x (76,533 nuclei) and Smart-seq (49,495 nuclei) sets.

## When NOT to use

- Query data are disease snRNA-seq profiles — the paper notes disease expression deviates substantially from normal data and recommends users train on their own disease data rather than the pre-trained atlas models
- The goal is annotation of non-human species — transfer of SELINA to other species is anticipated but not demonstrated in the paper
- Cell types of interest have fewer than 1,000 cells in the query dataset, since the benchmark restricts to types with >1,000 cells

## Inputs

- Allen Brain Atlas snRNA-seq 10x query dataset (76,533 nuclei)
- Allen Brain Atlas snRNA-seq Smart-seq query dataset (49,495 nuclei)
- SELINA pre-trained reference model files (pre-trained_params.pt, pre-trained_meta.pkl)
- Reference training data for selina train (plain/h5/mtx gene-by-cell matrices)

## Outputs

- Filtered query datasets restricted to the 19 cell types with >1,000 cells
- Per-cell predicted cell types and prediction probabilities (query_predictions.txt, query_probability.txt)
- Accuracy and MacroF1 per tool per dataset across 19 cell types
- Average metrics across the two Allen datasets and a ranking of the 9 annotation tools comparable to Figure 5B

## How to apply

Download the two Allen Brain Atlas snRNA-seq query datasets (76,533-nucleus 10x and 49,495-nucleus Smart-seq) and restrict both to the 19 cell types with more than 1,000 cells. Preprocess each query dataset with `selina preprocess`, which normalizes expression profiles, converts genes to hg38 symbol names, and performs PCA (30 PCs), UMAP, and Louvain clustering (resolution 0.6) with default QC cutoffs (count 1000, gene 500, cell 10). Run `selina predict` against the pre-trained reference model (pre-trained_params.pt / pre-trained_meta.pkl) with --prob-cutoff 0.9 to obtain per-cell predictions and probabilities. Run the 8 comparison annotation tools (Garnett, SCINA, scmap, scPred, SingleR, CHETAH, SingleCellNet, plus one marker/reference-based comparator) on the same filtered datasets using their reference-based or marker-based protocols, compute accuracy and MacroF1 per tool per dataset across the 19 cell types, average across the two datasets, and rank the 9 tools against the reported Figure 5B values.

## Related tools

- **SELINA** (Preprocess query data (normalize, hg38 symbol conversion, PCA/UMAP/Louvain) and predict cell types with pre-trained models) — https://github.com/SELINA-team/SELINA.py
- **Python** (Environment for running the selina CLI pipeline (preprocess/train/predict)) — https://github.com/SELINA-team/SELINA.py
- **R** (Runtime for comparator annotation tools and Seurat-based preprocessing outputs (query_res.rds)) — https://github.com/SELINA-team/SELINA.py
- **Garnett** (Marker-based comparator annotation tool benchmarked on the same filtered Allen datasets)
- **SCINA** (Marker-based comparator annotation tool benchmarked on the same filtered Allen datasets)
- **scmap** (Reference-based comparator annotation tool benchmarked on the same filtered Allen datasets)
- **scPred** (Reference-based comparator annotation tool benchmarked on the same filtered Allen datasets)
- **SingleR** (Reference-based comparator annotation tool benchmarked on the same filtered Allen datasets)
- **CHETAH** (Reference-based comparator annotation tool benchmarked on the same filtered Allen datasets)
- **SingleCellNet** (Reference-based comparator annotation tool benchmarked on the same filtered Allen datasets)

## Examples

```
selina preprocess --format plain --matrix ./query/allen_10x/query.txt --directory ./res/ ; selina predict --query-expr ./res/query_expr.txt --model ./res/pre-trained_params.pt --seurat ./res/query_res.rds --path-out ./res/predict/allen
```

## Evaluation signals

- Reproduced tool ranking matches Figure 5B ordering, with SELINA second of 9 tools
- SELINA average accuracy across the two Allen datasets is within tolerance of 93.32%
- SELINA average MacroF1 is within tolerance of 0.9336
- Both query datasets are restricted to the same 19 cell types with >1,000 cells before scoring
- Prediction files (query_predictions.txt, query_probability.txt) are produced for both the 10x and Smart-seq datasets with --prob-cutoff 0.9

## Limitations

- The pre-trained reference models cover 35 normal tissues, so tissue-specific dataset and cell-type abundance imbalances remain despite the atlas's 1.7 million cells / 230 cell types
- No comprehensive reference atlas with uniformed cell types is currently available due to the huge amount of public scRNA-seq data, so atlas coverage of rare types may be incomplete
- SELINA's Allen Brain Atlas validation is specific to harmonized snRNA-seq; results may not transfer to other species or unharmonized datasets

## Evidence

- [discussion] We also confirmed that SELINA can learn to classify cells from other unified databases, which was validated using data from the Allen Brain Atlas.: "We also confirmed that SELINA can learn to classify cells from other unified databases, which was validated using data from the Allen Brain Atlas."
- [methods] selina predict --query-expr ./res/preprocess/example1/example1_expr.txt --model ./res/pre-train/pre-trained_params.pt: "selina predict --query-expr ./res/preprocess/example1/example1_expr.txt  --model ./res/pre-train/pre-trained_params.pt"
- [methods] QC defaults: count cutoff 1000, gene cutoff 500, cell cutoff 10: "--count-cutoff COUNT_CUTOFF
                        Cutoff for the number of count in each cell. DEFAULT:
                        1000."
- [readme] pre-trained models are available on GitHub as SELINA models: [evidence span withheld — anchor status: external_doc]
- [intro] Marker-based and reference-based comparator methods listed: "Marker-based methods such as Garnett1 and SCINA2 rely on clustering performance and the quality of cell-type-specific marker genes."
- [readme] Query preprocessing outputs include Seurat object and expression matrix: "`query_res.rds`: a seurat object storing the normalized data, dimension reduction and clustering results"
