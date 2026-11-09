---
name: disease-scrnaseq-cell-type-annotation
description: Use when annotating cell types in a disease scRNA-seq query (e.g., type
  2 diabetes, NSCLC, Alzheimer's) against a normal-cell reference where batch/treatment
  effects make standard label transfer unreliable.
when_to_use_negative:
- The query dataset is normal tissue rather than disease — use SELINA's normal mode
  with the 35-tissue normal models and the fine-tuning step instead
- No pre-trained disease model exists for the disease condition of interest — the
  README notes disease datasets are extremely diverse across patients/groups and recommends
  users train on their own data via 'selina train'
- The task is de novo clustering or marker-based annotation without a reference atlas
  — marker-based tools like Garnett or SCINA follow a different paradigm than reference-based
  label transfer
edam_operation: http://edamontology.org/operation_1777
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: SELINA
  role: 'Disease-mode annotation framework: preprocess query, load pre-trained disease
    models, predict with --disease flag skipping fine-tuning'
  repo: https://github.com/SELINA-team/SELINA.py
- name: SELINA-reference
  role: Repository of pre-trained disease models (NSCLC, T2D, Alzheimer's) with parameters,
    reference genes, cell types, and cell sources
  repo: https://github.com/SELINA-team/SELINA-reference
- name: scmap
  role: Comparison tool run with default label-transfer procedure against the normal-cell
    reference
- name: SingleR
  role: Comparison tool run with default label-transfer procedure against the normal-cell
    reference
- name: CHETAH
  role: Comparison tool run with default label-transfer procedure against the normal-cell
    reference
- name: SingleCellNet
  role: Comparison tool run with default label-transfer procedure against the normal-cell
    reference
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.crmeth.2023.100577
    title: Single-cell assignment using multiple-adversarial domain adaptation network
      with large-scale references
  source_content_hash: sha256:6e5093f538948d41e494d0801732af844f4d1ea0d7ca10c5557e95305d5a6030
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
  - Python
  - R
  - SELINA
  - SELINA-reference
  - SingleCellNet
  - SingleR
  - scmap
  edam_operation: http://edamontology.org/operation_1777
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# disease-scRNAseq-cell-type-annotation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Annotate cell types in disease scRNA-seq query datasets (T2D, NSCLC, Alzheimer's) using SELINA's pre-trained disease models in disease mode, which skips fine-tuning because disease expression profiles deviate from normal references, and benchmark against scmap, SingleR, CHETAH, and SingleCellNet.

## When to use

Use when annotating cell types in a disease scRNA-seq query (e.g., type 2 diabetes, NSCLC, Alzheimer's) against a normal-cell reference where batch/treatment effects make standard label transfer unreliable.

## When NOT to use

- The query dataset is normal tissue rather than disease — use SELINA's normal mode with the 35-tissue normal models and the fine-tuning step instead
- No pre-trained disease model exists for the disease condition of interest — the README notes disease datasets are extremely diverse across patients/groups and recommends users train on their own data via 'selina train'
- The task is de novo clustering or marker-based annotation without a reference atlas — marker-based tools like Garnett or SCINA follow a different paradigm than reference-based label transfer

## Inputs

- Disease query scRNA-seq count matrix in plain, h5, or mtx format (gene-by-cell matrix for plain format)
- Paired pre-trained disease model files from SELINA-reference (e.g., T2D_params.pt and T2D_meta.pkl storing parameters, reference genes, cell types, and cell sources)

## Outputs

- query_predictions.txt: predicted cell type per query cell
- query_probability.txt: per-cell probability across reference cell types
- query_cellsources.txt: predicted cell source per cell (disease mode only)
- query_res.rds: Seurat object with normalized data, dimension reduction, and clustering
- Accuracy comparisons across the five tools and disease-stage-stratified correct-prediction rates

## How to apply

Preprocess each disease query dataset (T2D, NSCLC, Alzheimer's) with 'selina preprocess' using the documented defaults (count-cutoff 1000, gene-cutoff 500, cell-cutoff 10, 2000 variable genes, 30 PCs, cluster resolution 0.6), producing query_res.rds and query_expr.txt. Load the paired pre-trained disease models (e.g., T2D_params.pt and T2D_meta.pkl) from the SELINA-reference repository, which store model parameters plus reference genes, cell types, and cell sources. Run 'selina predict --disease --prob-cutoff 0.9', which skips the autoencoder fine-tuning step (justified because disease expression profiles deviate substantially from normal data) and additionally outputs per-cell cell-source predictions. For benchmarking, run scmap, SingleR, CHETAH, and SingleCellNet with their default label-transfer procedures against the normal-cell reference, then sweep the unknown-cell probability cutoff and compute annotation accuracy per tool, stratifying correct-prediction rates by disease stage.

## Related tools

- **SELINA** (Disease-mode annotation framework: preprocess query, load pre-trained disease models, predict with --disease flag skipping fine-tuning) — https://github.com/SELINA-team/SELINA.py
- **SELINA-reference** (Repository of pre-trained disease models (NSCLC, T2D, Alzheimer's) with parameters, reference genes, cell types, and cell sources) — https://github.com/SELINA-team/SELINA-reference
- **scmap** (Comparison tool run with default label-transfer procedure against the normal-cell reference)
- **SingleR** (Comparison tool run with default label-transfer procedure against the normal-cell reference)
- **CHETAH** (Comparison tool run with default label-transfer procedure against the normal-cell reference)
- **SingleCellNet** (Comparison tool run with default label-transfer procedure against the normal-cell reference)

## Examples

```
selina predict --query-expr ./res/query_expr.txt --model ./res/pre-trained_params.pt --seurat ./res/query_res.rds --path-out ./res/predict/disease --disease
```

## Evaluation signals

- On the T2D dataset, SELINA must rank first out of the five tools with accuracy near 1, ahead of ACTINN, mtSC, scmap, and SingleR
- Sweeping the unknown-cell probability cutoff must show SELINA's accuracy across the three disease benchmarks ranking first of the five tools (Figure 6A-6F)
- Stratifying correct-prediction rates by disease stage must reproduce the monotonic decline in accuracy for each tool (Figure 6G-6I)
- Disease-mode prediction must emit the extra query_cellsources.txt file in addition to the standard four prediction outputs
- Preprocessing must yield the four expected files (query_res.rds, query_expr.txt, query_cluster.png, query_cluster_DiffGenes.tsv)

## Limitations

- Disease models only cover non-small-cell lung carcinoma, type 2 diabetes, and Alzheimer's disease; the README recommends users train their own disease models since disease datasets are extremely diverse across patients and groups
- No comprehensive database for disease scRNA-seq data can be built due to diverse abnormal-cell expression profiles (e.g., pancreatic beta cells behaving differently across datasets)
- Accuracy declines monotonically with advancing disease stage for all tools, so late-stage samples will have lower correct-prediction rates

## Evidence

- [methods] Disease-mode prediction skips fine-tuning: [evidence span withheld — anchor status: external_doc]
- [results] T2D benchmark result: [evidence span withheld — anchor status: not_found]
