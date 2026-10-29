---
name: induced-vs-uninduced-logfc-comparison
description: Use when a controlled induction experiment (e.g. doxycycline-inducible
  miRNA in scRNA-seq) yields induced vs uninduced cell profiles and you must quantify
  whether a miRNA-inference method recovers the induction as logFC with significance.
when_to_use_negative:
- The goal is target-based miRNA activity scoring without a benchmark induction signal
  — miReact-style cross-gene-correlation methods are the appropriate control, not
  this logFC verification
- The pre-trained model's training context differs substantially from the application
  context and no matched cohort is available, since performance loss is expected (Figure
  S9)
- Profiles are so sparse that the pre-trained model fails and no retraining on detected
  genes is feasible
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0659
tools:
- name: miRSCAPE
  role: xgboost-based model applied to pooled and single-cell profiles to predict
    miRNA expression whose induced-vs-uninduced logFC is computed
  repo: https://github.com/hannenhalli-lab/miRSCAPE
- name: miReact
  role: cross-gene-correlation-based target-only control method run on the same induced/uninduced
    profiles to show it fails to detect induction
- name: Seurat
  role: Seurat object storage and modifySeuratObject-based preparation of scRNA data
    before prediction
- name: xgboost
  role: gradient boosting backend of the pre-trained miRSCAPE prediction models
- name: R 4.1.0
  role: runtime environment (with Seurat and xgboost dependencies) in which the comparison
    is executed
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.isci.2022.104962
    title: miRSCAPE - inferring miRNA expression from scRNA-seq data
  source_content_hash: sha256:90a725534c6aae8ce05745843b61864458f381f7dec99d01499322921088efec
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/hannenhalli-lab/miRSCAPE
  repo_url: https://github.com/hannenhalli-lab/miRSCAPE
  tools:
  - R
  - R 4.1.0
  - Seurat
  - miRSCAPE
  - miReact
  - xgboost
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0659
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# induced-vs-uninduced-logfc-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Compares predicted miRNA expression between experimentally induced and uninduced single-cell samples by computing induced-vs-uninduced log-fold changes and Wilcoxon p-values, used to verify that miRSCAPE recapitulates miR-199a-5p/3p induction in scRNA-seq data.

## When to use

Use when a controlled induction experiment (e.g. doxycycline-inducible miRNA in scRNA-seq) yields induced vs uninduced cell profiles and you must quantify whether a miRNA-inference method recovers the induction as logFC with significance.

## When NOT to use

- The goal is target-based miRNA activity scoring without a benchmark induction signal — miReact-style cross-gene-correlation methods are the appropriate control, not this logFC verification
- The pre-trained model's training context differs substantially from the application context and no matched cohort is available, since performance loss is expected (Figure S9)
- Profiles are so sparse that the pre-trained model fails and no retraining on detected genes is feasible

## Inputs

- Seurat object with induced and uninduced single-cell RNA-seq expression data (miR-199a induction experiment)
- Pre-trained CCLE-based miRSCAPE xgboost models for miR-199a-5p and miR-199a-3p
- Cluster/cell-type assignments for pooling cells

## Outputs

- Per-case induced-vs-uninduced log-fold changes and Wilcoxon p-values for pooled miRSCAPE, single-cell miRSCAPE, and miReact strategies
- mir199a_logfc_results.csv with one row per miRNA/cell-type/strategy comparison

## How to apply

Pool cells within each cluster to obtain non-sparse transcriptomic profiles (sparse single-cell profiles can break pre-trained xgboost models), then apply the CCLE-trained miRSCAPE models to both induced and uninduced pooled profiles and, as a sparser variant, to individual cells. Optionally run miReact on the same profiles as a target-only control. For each miRNA and cell type, compute the induced-vs-uninduced log-fold change of predicted values and a Wilcoxon rank-sum p-value. Judge success against the reported benchmarks: pooled miRSCAPE logFC 6.2, 6.0, 5.9, 4.9 (p < E-72); single-cell miRSCAPE logFC 2.6, 2.6, 1.9, 1.5 (p < E-36); miReact control logFC 0.33, 0.15, -1.00, -1.13, and write all rows to mir199a_logfc_results.csv.

## Related tools

- **miRSCAPE** (xgboost-based model applied to pooled and single-cell profiles to predict miRNA expression whose induced-vs-uninduced logFC is computed) — https://github.com/hannenhalli-lab/miRSCAPE
- **miReact** (cross-gene-correlation-based target-only control method run on the same induced/uninduced profiles to show it fails to detect induction)
- **Seurat** (Seurat object storage and modifySeuratObject-based preparation of scRNA data before prediction)
- **xgboost** (gradient boosting backend of the pre-trained miRSCAPE prediction models)
- **R 4.1.0** (runtime environment (with Seurat and xgboost dependencies) in which the comparison is executed)

## Examples

```
denem = modifySeuratObject(pbmc = example, clusterId = clustId); pred = miRSCAPE(bulkmRNA = bulkk_mrna, bulkmiRNA = bulkk_mirna, scmRNA = denem)
```

## Evaluation signals

- Pooled miRSCAPE reproduces logFC ≈ 6.2, 6.0, 5.9, 4.9 for the 2 miRNAs × 2 cell types with Wilcoxon p < E-72
- Unpooled single-cell miRSCAPE reproduces logFC ≈ 2.6, 2.6, 1.9, 1.5 with p < E-36
- miReact control yields near-zero or negative logFC (0.33, 0.15, -1.00, -1.13), confirming specificity of miRSCAPE's induction signal
- All induced-vs-uninduced comparisons are significantly positive (all 4 cases significant), and results are written to mir199a_logfc_results.csv

## Limitations

- Single-cell (unpooled) prediction yields markedly smaller logFC (1.5–2.6 vs 4.9–6.2) because of sparsity/dropout
- Given the large dropout rate, a pre-trained model may not work and miRSCAPE may need to be retrained using only genes detected at a reasonable level
- Accuracy is reduced when the cellular/tissue context of the application sample does not match the training cohort (e.g. CCLE bulk training vs scRNA application)

## Evidence

- [other] Pooled logFC benchmark: [evidence span withheld — anchor status: not_found]
