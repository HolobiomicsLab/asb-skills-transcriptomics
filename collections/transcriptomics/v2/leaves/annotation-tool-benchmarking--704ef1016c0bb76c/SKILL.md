---
name: annotation-tool-benchmarking
description: Use when a study claims a cell-annotation tool's ity and the claim must
  be reproduced on the same query datasets (e.g. Allen Brain Atlas snRNA-seq, T2D/NSCLC/AD
  disease data) with ranked accuracy/MacroF1 metrics.
when_to_use_negative:
- The query data are non-human or species other than human — SELINA's atlas covers
  230 distinct human cell types and transfer to other species is only anticipated,
  not demonstrated
- The query dataset is a patient-specific disease sample whose expression profile
  deviates substantially from available disease models — the SELINA README recommends
  users train their own disease models since 'disease datasets are extremely diverse
  for different patients and groups'
- The task is clustering or dimensionality reduction itself rather than supervised
  annotation benchmarking — no reference labels are needed for such tasks
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: SELINA
  role: Primary annotation tool being benchmarked; preprocess/train/predict CLI with
    prob-cutoff 0.9 and optional --disease mode
  repo: https://github.com/SELINA-team/SELINA.py
- name: SELINA-reference
  role: Source of the pre-trained normal (35 tissues) and disease (NSCLC, T2D, AD)
    models used as the annotation references
  repo: https://github.com/SELINA-team/SELINA-reference
- name: Garnett
  role: Marker-based comparator annotation tool run on the same filtered datasets
- name: SCINA
  role: Marker-based comparator annotation tool run on the same filtered datasets
- name: scmap
  role: Reference-based comparator label-transfer tool run on the same query datasets
- name: scPred
  role: Reference-based comparator annotation tool in the 9-tool Allen Brain Atlas
    benchmark
- name: SingleR
  role: Reference-based comparator label-transfer tool run with default procedure
- name: CHETAH
  role: Reference-based comparator annotation tool in both Allen Atlas and disease
    benchmarks
- name: SingleCellNet
  role: Reference-based comparator annotation tool run with default label-transfer
    procedure
- name: presto
  role: Fast differential expression/marker discovery used within SELINA's processing
    pipeline
  repo: https://github.com/immunogenomics/presto
provenance:
  source_task_ids:
  - task_004
  - task_005
  source_papers:
  - doi: 10.1016/j.crmeth.2023.100577
    title: Single-cell assignment using multiple-adversarial domain adaptation network
      with large-scale references
  source_content_hash: sha256:89829fb24b3f7ab52ba6bf634a56a1a5de5665df4f6c56a6015d7410e24b122e
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
  - SELINA-reference
  - SingleCellNet
  - SingleR
  - presto
  - scPred
  - scmap
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.65
n_claims: 20
---

# annotation-tool-benchmarking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark single-cell RNA-seq cell-type annotation tools by running each tool's documented reference-based or marker-based protocol on identical query datasets and comparing accuracy and MacroF1 against the published rankings (e.g. SELINA 93.32% accuracy, MacroF1 0.9336 on Allen Brain Atlas data).

## When to use

Use when a study claims a cell-annotation tool's superiority and the claim must be reproduced on the same query datasets (e.g. Allen Brain Atlas snRNA-seq, T2D/NSCLC/AD disease data) with ranked accuracy/MacroF1 metrics.

## When NOT to use

- The query data are non-human or species other than human — SELINA's atlas covers 230 distinct human cell types and transfer to other species is only anticipated, not demonstrated
- The query dataset is a patient-specific disease sample whose expression profile deviates substantially from available disease models — the SELINA README recommends users train their own disease models since 'disease datasets are extremely diverse for different patients and groups'
- The task is clustering or dimensionality reduction itself rather than supervised annotation benchmarking — no reference labels are needed for such tasks

## Inputs

- Query snRNA-seq/scRNA-seq expression matrices in plain, h5, or mtx format (e.g. Allen Brain Atlas 76,533-nucleus 10x and 49,495-nucleus Smart-seq datasets; T2D, NSCLC, AD disease datasets)
- Cell-level ground-truth labels for the query cells (19 Allen Atlas cell types with >1,000 cells)
- Pre-trained SELINA models (normal and disease: NSCLC, T2D, AD) with *_params.pt and *_meta.pkl files
- Reference datasets for the comparator tools (scmap, SingleR, CHETAH, SingleCellNet, scPred, Garnett, SCINA, ACTINN, mtSC)

## Outputs

- query_predictions.txt (per-cell predicted cell types)
- query_probability.txt (per-cell probabilities over reference cell types)
- query_cellsources.txt (per-cell predicted cell source, disease mode only)
- Ranked per-tool table of average accuracy and MacroF1 across datasets, comparable to Figure 5B and Figure 6A-6I
- Disease-stage stratified correct-prediction rate curves

## How to apply

Download the query datasets and apply any dataset-specific restrictions first (e.g. restrict the 76,533-nucleus 10x and 49,495-nucleus Smart-seq Allen Brain Atlas datasets to the 19 cell types with >1,000 cells). Preprocess every query identically with `selina preprocess` using default QC cutoffs (count 1000, gene 500, cell 10), 2000 variable genes, 30 PCs, UMAP, and Louvain clustering at resolution 0.6. Train/apply each annotation tool under comparable settings: SELINA with `selina train` (producing pre-trained_params.pt and pre-trained_meta.pkl) and `selina predict --prob-cutoff 0.9`; in disease mode add `--disease` to skip fine-tuning and also emit per-cell cell-source predictions. Run the comparators (Garnett, SCINA, scmap, scPred, SingleR, CHETAH, SingleCellNet, ACTINN, mtSC) using each tool's own reference-based or marker-based protocol on the same filtered data. Sweep the unknown-cell probability cutoff, compute per-tool accuracy and MacroF1 per dataset, average across datasets, and rank tools against the reported figure values (SELINA second of 9 on Allen Atlas; first of 5 with accuracy near 1 on T2D; monotonic accuracy decline stratified by disease stage).

## Related tools

- **SELINA** (Primary annotation tool being benchmarked; preprocess/train/predict CLI with prob-cutoff 0.9 and optional --disease mode) — https://github.com/SELINA-team/SELINA.py
- **SELINA-reference** (Source of the pre-trained normal (35 tissues) and disease (NSCLC, T2D, AD) models used as the annotation references) — https://github.com/SELINA-team/SELINA-reference
- **Garnett** (Marker-based comparator annotation tool run on the same filtered datasets)
- **SCINA** (Marker-based comparator annotation tool run on the same filtered datasets)
- **scmap** (Reference-based comparator label-transfer tool run on the same query datasets)
- **scPred** (Reference-based comparator annotation tool in the 9-tool Allen Brain Atlas benchmark)
- **SingleR** (Reference-based comparator label-transfer tool run with default procedure)
- **CHETAH** (Reference-based comparator annotation tool in both Allen Atlas and disease benchmarks)
- **SingleCellNet** (Reference-based comparator annotation tool run with default label-transfer procedure)
- **presto** (Fast differential expression/marker discovery used within SELINA's processing pipeline) — https://github.com/immunogenomics/presto

## Examples

```
selina predict --query-expr ./res/query_expr.txt --model ./res/pre-trained_params.pt --seurat ./res/query_res.rds --path-out ./res/predict/disease --disease
```

## Evaluation signals

- Reproduced Allen Brain Atlas table matches Figure 5B ordering: SELINA second of 9 tools with average accuracy within tolerance of 93.32% and MacroF1 within tolerance of 0.9336
- T2D benchmark reproduces SELINA's accuracy near 1 and first-place rank among five tools (SELINA, ACTINN, mtSC, scmap, SingleR)
- Disease-stage stratified accuracy curves show the reported monotonic decline in correct predictions for each tool (Figure 6G-6I)
- Preprocessing outputs are complete: query_res.rds Seurat object, query_expr.txt matrix, UMAP cluster PNG, and per-cluster DEG TSV from `selina preprocess`
- Prediction outputs include query_predictions.txt and query_probability.txt, plus query_cellsources.txt only when --disease mode is used

## Limitations

- Disease prediction lacks fine-tuning (the --disease flag skips it), so accuracy depends heavily on how well the chosen pre-trained disease model matches the patient cohort; the README states users should use their own data for disease models
- Reference update to include more experimentally validated cell types is planned, not implemented; graph-based structural/explainability algorithms are also future work
- Disease scRNA-seq profiles vary across patients and groups (e.g. pancreatic beta cells behaving differently across datasets), making a comprehensive disease reference database infeasible

## Evidence

- [results] Allen benchmark reproduced ranking: [evidence span withheld — anchor status: not_found]
- [methods] Preprocess QC defaults: "Quality control arguments:
--count-cutoff COUNT_CUTOFF
                      Cutoff for the number of count in each cell. DEFAULT:
                      1000."
- [readme] Disease mode skips fine-tuning: [evidence span withheld — anchor status: external_doc]
- [discussion] Allen Atlas validation of cross-database learning: "We also confirmed that SELINA can learn to classify cells from other unified databases, which was validated using data from the Allen Brain Atlas."
- [intro] Benchmark motivation vs mainstream methods: "the benchmark in multiple disease scenarios demonstrated that SELINA could annotate cell types with a higher accuracy than the mainstream methods"
