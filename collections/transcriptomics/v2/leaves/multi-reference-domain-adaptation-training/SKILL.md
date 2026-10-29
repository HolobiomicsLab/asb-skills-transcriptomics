---
name: multi-reference-domain-adaptation-training
description: Use when a query scRNA-seq dataset must be annotated against multiple
  reference datasets simultaneously, or when reproducing the leave-one-dataset-out
  benchmark of SELINA against comparator annotation tools.
when_to_use_negative:
- Input is a single query dataset already paired with a single matching reference
  — standard single-reference annotation tools suffice
- Predicting on disease query data where no suitable disease model exists — the paper
  notes it is 'difficult to build a comprehensive database for disease scRNA-seq data'
  and recommends users train on their own disease data
- The query data are non-human scRNA-seq — transfer to other species is anticipated
  but not demonstrated
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3474
tools:
- name: SELINA
  role: Framework performing cell-type balancing (SMOTE), MADA pre-training, and autoencoder
    fine-tuning via 'selina train'
  repo: https://github.com/SELINA-team/SELINA.py
- name: SELINA-reference
  role: Source of pre-trained model files (*_params.pt, *_meta.pkl) and curated reference
    datasets for the 35 normal tissues and 3 disease conditions
  repo: https://github.com/SELINA-team/SELINA-reference
- name: presto
  role: R dependency installed alongside SELINA (via devtools) used within the SELINA
    preprocessing/pre-training environment
  repo: https://github.com/immunogenomics/presto
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.crmeth.2023.100577
    title: Single-cell assignment using multiple-adversarial domain adaptation network
      with large-scale references
  source_content_hash: sha256:b14dbb76b92838b63276b4846c91c1263e1166c5d2fd6ec3101f3871d6750105
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
  - CellTypist
  - Python
  - R
  - SELINA
  - SELINA-reference
  - Seurat
  - SingleCellNet
  - SingleR
  - mtSC
  - presto
  - scmap
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5333333333333333
n_claims: 15
---

# multi-reference-domain-adaptation-training

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Train a SELINA cell-type annotation model on multiple curated reference datasets (e.g., GSE*_expr.txt / GSE*_meta.txt pairs) using SMOTE cell-type balancing, multi-adversarial domain adaptation (MADA) pre-training, and autoencoder fine-tuning, producing *_params.pt and *_meta.pkl model files.

## When to use

Use when a query scRNA-seq dataset must be annotated against multiple reference datasets simultaneously, or when reproducing the leave-one-dataset-out benchmark of SELINA against comparator annotation tools.

## When NOT to use

- Input is a single query dataset already paired with a single matching reference — standard single-reference annotation tools suffice
- Predicting on disease query data where no suitable disease model exists — the paper notes it is 'difficult to build a comprehensive database for disease scRNA-seq data' and recommends users train on their own disease data
- The query data are non-human scRNA-seq — transfer to other species is anticipated but not demonstrated

## Inputs

- Curated reference expression matrices (GSE*_expr.txt, plain/mtx/h5 formats)
- Reference meta files with Sample, Patient, Platform, and CellType annotations (GSE*_meta.txt)
- Multi-reference training directory (normal or disease condition)

## Outputs

- pre-trained_params.pt (all trained model parameters)
- pre-trained_meta.pkl (cell types, genes, and cell sources if in disease mode)

## How to apply

Run 'selina train' on the reference directory (e.g., --path-in ./reference_data/ --path-out ./res/pre-train/ --outprefix pre-trained), where each dataset contributes a plain/h5/mtx expression matrix plus meta file with CellType labels. The algorithm first balances cell types by SMOTE synthetic oversampling of rare cell types, then pre-trains a supervised deep-learning classifier with MADA to remove batch effects across reference datasets, and finally fine-tunes with an autoencoder that adapts parameters to the query data distribution. For disease-mode models (non-small-cell lung carcinoma, type 2 diabetes, Alzheimer's disease), the architecture and cost function differ slightly and the fine-tuning step is removed at predict time. Output is a pre-trained_params.pt file holding all model parameters and a pre-trained_meta.pkl holding cell types (plus cell sources in disease mode) and genes. For the leave-one-dataset-out benchmark, retrain on the remaining reference datasets for each held-out tissue split.

## Related tools

- **SELINA** (Framework performing cell-type balancing (SMOTE), MADA pre-training, and autoencoder fine-tuning via 'selina train') — https://github.com/SELINA-team/SELINA.py
- **SELINA-reference** (Source of pre-trained model files (*_params.pt, *_meta.pkl) and curated reference datasets for the 35 normal tissues and 3 disease conditions) — https://github.com/SELINA-team/SELINA-reference
- **presto** (R dependency installed alongside SELINA (via devtools) used within the SELINA preprocessing/pre-training environment) — https://github.com/immunogenomics/presto

## Examples

```
selina train --path-in ./reference/normal_data --path-out ./res/
```

## Evaluation signals

- Training emits Fine-tuning1 (50 iterations) and Fine-tuning2 (10 iterations) progress bars; both must complete without error
- Exactly two output files are produced: pre-trained_params.pt and pre-trained_meta.pkl; the meta pickle must contain the reference cell-type list and gene list
- In leave-one-dataset-out benchmarking, the trained model should reproduce SELINA's reported accuracy/MacroF1 (61.51%/0.5126 single-platform, 64.42%/0.5082 multi-platform) within tolerance
- The reference atlas used for training spans 1.7 million cells across 230 human cell types; per-split reference counts should be consistent with the curated HUSCH datasets
- Downstream 'selina predict' with --prob-cutoff 0.9 on a held-out query must yield per-cell predictions and per-cell-type probability files for every split

## Limitations

- The reference is imbalanced in tissue-specific datasets and cell-type abundance despite its large scale
- Disease models only cover non-small-cell lung carcinoma, type 2 diabetes, and Alzheimer's disease; users are advised to train on their own disease data since disease datasets are extremely diverse across patients and groups
- SMOTE oversampling is needed because minority cell types are otherwise ignored during modeling; very rare types may still be poorly represented

## Evidence

- [readme] selina train --path-in ./reference/normal_data --path-out ./res/: [evidence span withheld — anchor status: external_doc]
