---
name: cross-method-annotation-benchmarking
description: Use when the goal is to quantitatively compare cell-type annotation tools
  on the same reference/query dataset splits, reproducing accuracy and MacroF1 tables
  like SELINA vs ACTINN, SingleCellNet, mtSC, CellTypist, Cell BLAST, SingleR, scmap,
  Seurat, SVM, RF, kNN.
when_to_use_negative:
- Query data is from a disease context not covered by the disease models (which only
  cover non-small-cell lung carcinoma, type 2 diabetes, and Alzheimer's) — the README
  recommends users train on their own data since 'the disease datasets are extremely
  diverse for different patients and groups'
- The task is annotating non-human species — transfer of SELINA to other species is
  anticipated but was not demonstrated
- Data are not human scRNA-seq expression profiles with cell-type labels (SELINA's
  reference atlas covers 1.7M cells / 230 human cell types across 35 tissues and 7
  platforms)
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: SELINA
  role: Framework under benchmark; its 'selina train', 'selina preprocess', and 'selina
    predict' commands produce the annotations evaluated on each leave-one-dataset-out
    split
  repo: https://github.com/SELINA-team/SELINA.py
- name: SingleCellNet
  role: Comparator method run on the same reference/query splits with default recommended
    settings
- name: ACTINN
  role: Comparator method run on the same reference/query splits with default recommended
    settings
- name: mtSC
  role: Comparator method run on the same reference/query splits with default recommended
    settings
- name: CellTypist
  role: Comparator method run on the same reference/query splits with default recommended
    settings
- name: Cell BLAST
  role: Comparator method run on the same reference/query splits with default recommended
    settings
- name: SingleR
  role: Comparator method run on the same reference/query splits with default recommended
    settings
- name: scmap
  role: Comparator method run on the same reference/query splits with default recommended
    settings
- name: Seurat
  role: Transfer-based comparator (dependency r-seurat=4.0.3 in the SELINA environment)
    run on the same splits
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.crmeth.2023.100577
    title: Single-cell assignment using multiple-adversarial domain adaptation network
      with large-scale references
  source_content_hash: sha256:3db4c5c202a64338681972371eb36f1fa64a8b0eaa09f8aed7277798410867e3
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
  - ACTINN
  - Cell BLAST
  - CellTypist
  - Python
  - R
  - SELINA
  - Seurat
  - SingleCellNet
  - SingleR
  - mtSC
  - scmap
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7916666666666666
n_claims: 24
---

# cross-method-annotation-benchmarking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark a cell-type annotation framework (SELINA) against 13 reference-based and machine-learning comparators using leave-one-dataset-out splits over single-platform and multi-platform human tissue scRNA-seq datasets, scoring overall accuracy and MacroF1.

## When to use

Use when the goal is to quantitatively compare cell-type annotation tools on the same reference/query dataset splits, reproducing accuracy and MacroF1 tables like SELINA vs ACTINN, SingleCellNet, mtSC, CellTypist, Cell BLAST, SingleR, scmap, Seurat, SVM, RF, kNN.

## When NOT to use

- Query data is from a disease context not covered by the disease models (which only cover non-small-cell lung carcinoma, type 2 diabetes, and Alzheimer's) — the README recommends users train on their own data since 'the disease datasets are extremely diverse for different patients and groups'
- The task is annotating non-human species — transfer of SELINA to other species is anticipated but was not demonstrated
- Data are not human scRNA-seq expression profiles with cell-type labels (SELINA's reference atlas covers 1.7M cells / 230 human cell types across 35 tissues and 7 platforms)

## Inputs

- Curated per-dataset reference expression files (GSE*_expr.txt, gene-by-cell plain-format matrices)
- Curated per-dataset meta/label files (GSE*_meta.txt with CellType annotations)
- Pre-trained SELINA model files (*_params.pt and *_meta.pkl)
- Held-out query scRNA-seq expression matrices in plain, h5, or mtx format
- Comparator tool configurations (ACTINN, SingleCellNet, mtSC, scibet, CellTypist, SVM, Seurat, SingleR, RF, CelliD, scmap, kNN, Cell BLAST)

## Outputs

- Per-cell prediction files (query_predictions.txt) for every SELINA split
- Per-cell-type probability files (query_probability.txt) for every SELINA split
- Trained SELINA Base model files per leave-one-out split (pre-trained_params.pt, pre-trained_meta.pkl)
- Per-tissue accuracy and MacroF1 values per method, aggregated over 9 single-platform and 14 multi-platform tissues

## How to apply

Download the curated reference datasets (e.g. GSE*_expr.txt and GSE*_meta.txt, such as GSE139534_Lung-Adult, GSE123405_Lung-Adult, GSE130148_Lung-Adult) and pre-trained model files (*_params.pt, *_meta.pkl) from the SELINA-reference repository. For each leave-one-dataset-out split, run 'selina train' on the remaining reference datasets (SELINA's pipeline is cell type balancing with SMOTE, pre-training with multi-adversarial domain adaptation, then autoencoder fine-tuning), and use the full multi-reference model for the full configuration. For each held-out query dataset, run 'selina preprocess' with default QC (count-cutoff 1000, gene-cutoff 500, cell-cutoff 10, 2000 variable genes, 30 PCs) then 'selina predict' with --prob-cutoff 0.9 to emit per-cell predictions and per-cell-type probability files. Run each comparator (ACTINN, SingleCellNet, mtSC, scibet, CellTypist, SVM, Seurat, SingleR, RF, CelliD, scmap, kNN, Cell BLAST) on identical splits with default recommended settings, then compute overall accuracy and Macro F1 per tissue and per method, aggregating over the 9 single-platform and 14 multi-platform tissues. The rationale is that multi-reference training with SMOTE balancing and MADA-based batch-effect removal should generalize better than single-reference tools when labels transfer across datasets and platforms.

## Related tools

- **SELINA** (Framework under benchmark; its 'selina train', 'selina preprocess', and 'selina predict' commands produce the annotations evaluated on each leave-one-dataset-out split) — https://github.com/SELINA-team/SELINA.py
- **SingleCellNet** (Comparator method run on the same reference/query splits with default recommended settings)
- **ACTINN** (Comparator method run on the same reference/query splits with default recommended settings)
- **mtSC** (Comparator method run on the same reference/query splits with default recommended settings)
- **CellTypist** (Comparator method run on the same reference/query splits with default recommended settings)
- **Cell BLAST** (Comparator method run on the same reference/query splits with default recommended settings)
- **SingleR** (Comparator method run on the same reference/query splits with default recommended settings)
- **scmap** (Comparator method run on the same reference/query splits with default recommended settings)
- **Seurat** (Transfer-based comparator (dependency r-seurat=4.0.3 in the SELINA environment) run on the same splits)

## Examples

```
selina predict --query-expr ./res/query_expr.txt --model ./res/pre-trained_params.pt --seurat ./res/query_res.rds --path-out ./res/predict --prob-cutoff 0.9
```

## Evaluation signals

- Reproduced summary metrics match reported values within tolerance: SELINA 61.51% accuracy / 0.5126 MacroF1 on single-platform and 64.42% / 0.5082 on multi-platform evaluations
- SELINA ranks first with the highest average accuracy and MacroF1 across both the single-platform and multi-platform leave-one-dataset-out evaluations
- Per-cell prediction and probability files exist for every leave-one-out split (query_predictions.txt and query_probability.txt)
- Metrics are aggregated consistently over the same 9 single-platform tissues and 14 multi-platform tissues for every method
- Comparator methods are run on identical reference/query splits with their default recommended settings, ensuring a fair comparison

## Limitations

- Reference atlas remains imbalanced: 'the number of datasets in different tissues and the cell-type abundance in each tissue is still imbalanced'
- Disease benchmarking is limited to a few conditions; no comprehensive disease reference database exists because disease expression profiles are highly patient- and group-specific
- SELINA's minority-cell-type handling depends on SMOTE oversampling, and marker-based/cluster-dependent comparators may be disadvantaged by shared clustering steps

## Evidence

- [readme] Training command: [evidence span withheld — anchor status: external_doc]
- [readme] Disease-mode caveat: [evidence span withheld — anchor status: external_doc]
