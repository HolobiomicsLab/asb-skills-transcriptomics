---
name: leave-one-dataset-out-cross-validation
description: Use when comparing cell-type annotation methods on multiple reference
  datasets and you need an unbiased estimate of generalization to an entirely unseen
  dataset, using accuracy and MacroF1 per tissue.
when_to_use_negative:
- When the query is a disease dataset with abnormal expression profiles — disease
  models only cover non-small-cell lung carcinoma, type 2 diabetes and Alzheimer's
  disease, and fine-tuning is skipped in disease mode
- When the goal is annotating a single query from a single reference rather than estimating
  generalization across an atlas of datasets
- When the input is bulk transcriptomics or already an aggregated feature table rather
  than single-cell expression data
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3474
tools:
- name: SELINA
  role: Cell-type annotation framework being benchmarked; provides 'selina train',
    'selina preprocess' and 'selina predict' commands for each leave-one-dataset-out
    split
  repo: https://github.com/SELINA-team/SELINA-reference
- name: SingleCellNet
  role: Comparator reference-based annotation method run with default recommended
    settings on the same splits
- name: ACTINN
  role: Comparator reference-based annotation method run with default recommended
    settings on the same splits
- name: CellTypist
  role: Comparator reference-based annotation method run with default recommended
    settings on the same splits
- name: SingleR
  role: Comparator reference-based annotation method run with default recommended
    settings on the same splits
- name: scmap
  role: Comparator reference-based annotation method run with default recommended
    settings on the same splits
- name: Cell BLAST
  role: Comparator reference-based annotation method run with default recommended
    settings on the same splits
- name: Seurat
  role: Comparator method (and dependency of SELINA's preprocess/cluster step, r-seurat
    4.0.3) run with default recommended settings on the same splits
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.crmeth.2023.100577
    title: Single-cell assignment using multiple-adversarial domain adaptation network
      with large-scale references
  source_content_hash: sha256:e442c0225035aa426bc6c6e9a034034d588d68f4cf0cda2d13d60cdbd8b5ea32
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/SELINA-team/SELINA.py
  repo_url: https://github.com/SELINA-team/SELINA-reference
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
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6111111111111112
n_claims: 18
---

# leave-one-dataset-out-cross-validation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A benchmarking protocol that evaluates SELINA (and 13 comparator cell-type annotation tools) by holding out entire curated GEO datasets (e.g. GSE*_expr.txt/GSE*_meta.txt) as query sets, training on the remaining references, and scoring accuracy and MacroF1 across single- and multi-platform tissues.

## When to use

Use when comparing cell-type annotation methods on multiple reference datasets and you need an unbiased estimate of generalization to an entirely unseen dataset, using accuracy and MacroF1 per tissue.

## When NOT to use

- When the query is a disease dataset with abnormal expression profiles — disease models only cover non-small-cell lung carcinoma, type 2 diabetes and Alzheimer's disease, and fine-tuning is skipped in disease mode
- When the goal is annotating a single query from a single reference rather than estimating generalization across an atlas of datasets
- When the input is bulk transcriptomics or already an aggregated feature table rather than single-cell expression data

## Inputs

- Curated reference expression and metadata files per dataset (e.g. GSE*_expr.txt, GSE*_meta.txt) in plain format
- Pre-trained SELINA model files (*_params.pt and *_meta.pkl) from the SELINA-reference repository
- Held-out query expression matrix (gene-by-cell, plain/h5/mtx)
- 13 comparator tool installations (ACTINN, SingleCellNet, mtSC, scibet, CellTypist, SVM, Seurat, SingleR, RF, CelliD, scmap, kNN, Cell BLAST)

## Outputs

- Per-cell predicted cell-type files (query_predictions.txt) for every split
- Per-cell-type probability files (query_probability.txt) for every split
- Per-tissue and per-method accuracy and MacroF1 tables aggregated over 9 single-platform and 14 multi-platform tissues
- Ranked summary table (SELINA: 61.51% accuracy / 0.5126 MacroF1 single-platform; 64.42% / 0.5082 multi-platform)

## How to apply

For each leave-one-dataset-out split, download the curated expression/meta files and pre-trained models (*_params.pt, *_meta.pkl) from the SELINA-reference repository, then run 'selina train' on the remaining reference datasets (full multi-reference model for the full configuration). Preprocess the held-out query with default QC (count-cutoff 1000, gene-cutoff 500, cell-cutoff 10, 2000 variable genes, 30 PCs) and predict with --prob-cutoff 0.9 to produce per-cell predictions and per-cell-type probability files. Run each comparator (ACTINN, SingleCellNet, mtSC, scibet, CellTypist, SVM, Seurat, SingleR, RF, CelliD, scmap, kNN, Cell BLAST) on identical reference/query splits with their default recommended settings. Rationale: whole-dataset holdout mimics the real scenario of annotating a new dataset from a different batch/platform, exercising SELINA's multi-adversarial domain adaptation and SMOTE balancing rather than within-dataset random splits.

## Related tools

- **SELINA** (Cell-type annotation framework being benchmarked; provides 'selina train', 'selina preprocess' and 'selina predict' commands for each leave-one-dataset-out split) — https://github.com/SELINA-team/SELINA-reference
- **SingleCellNet** (Comparator reference-based annotation method run with default recommended settings on the same splits)
- **ACTINN** (Comparator reference-based annotation method run with default recommended settings on the same splits)
- **CellTypist** (Comparator reference-based annotation method run with default recommended settings on the same splits)
- **SingleR** (Comparator reference-based annotation method run with default recommended settings on the same splits)
- **scmap** (Comparator reference-based annotation method run with default recommended settings on the same splits)
- **Cell BLAST** (Comparator reference-based annotation method run with default recommended settings on the same splits)
- **Seurat** (Comparator method (and dependency of SELINA's preprocess/cluster step, r-seurat 4.0.3) run with default recommended settings on the same splits)

## Examples

```
selina preprocess --format plain --matrix ./query/normal_data/query.txt --directory ./res/ ; selina train --path-in ./reference/normal_data --path-out ./res/ ; selina predict --query-expr ./res/query_expr.txt --model ./res/pre-trained_params.pt --seurat ./res/query_res.rds --path-out ./res/predict/
```

## Evaluation signals

- Reproduced SELINA summary accuracy/MacroF1 match 61.51%/0.5126 (single-platform) and 64.42%/0.5082 (multi-platform) within a reasonable tolerance
- SELINA ranks first in accuracy and MacroF1 across both single- and multi-platform evaluations
- Per-cell prediction files (query_predictions.txt) and probability files (query_probability.txt) exist for every leave-one-dataset-out split
- MacroF1 and accuracy are computed per tissue and aggregated over 9 single-platform and 14 multi-platform tissues with no missing splits
- Each comparator runs on the exact same reference/query partition so that per-tissue scores are directly comparable across methods

## Limitations

- The reference remains imbalanced: 'the number of datasets in different tissues and the cell-type abundance in each tissue is still imbalanced'
- Disease-mode benchmarks are limited to three conditions (non-small-cell lung carcinoma, type 2 diabetes, Alzheimer's disease) since 'the disease datasets are extremely diverse for different patients and groups'
- Mostly human-focused; transfer to other species is anticipated but not demonstrated

## Evidence

- [readme] disease mode flag: [evidence span withheld — anchor status: external_doc]
