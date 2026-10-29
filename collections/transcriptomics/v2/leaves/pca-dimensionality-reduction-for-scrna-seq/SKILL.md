---
name: pca-dimensionality-reduction-for-scrna-seq
description: Use when a raw single-cell counts matrix (csv/rds/h5, genes as rows and
  cells as columns) must be reduced to a lower-dimensional gene space before cell
  clustering or neural-network cell-type annotation in a cancer TME scRNA-seq pipeline.
when_to_use_negative:
- Input is already a reduced/embedded representation (e.g., pre-computed PCA or cluster
  assignments) rather than a raw gene-by-cell counts matrix
- Data have not passed the scWizard quality-control filters (<200 genes, >10,000 genes,
  >5% mitochondrial reads), since low-quality cells distort the PCA space
- Tissue-specific non-TME cell types are the target — the article notes tissue-specific
  cells (e.g., alveolar cells in lung cancers) may land in the Unknown cluster
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: scWizard
  role: Packages the Seurat-standardized pipeline whose dimension-reduction step produces
    the PCA space used to size the deep neural network's input layer for cell annotation
  repo: https://github.com/Dulab2020/scWizard
- name: Seurat
  role: Provides the dimension reduction (with cell clustering and differential gene
    expression detection) that scWizard wraps as its standardized scRNA-seq pipeline
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.csbj.2022.08.028
    title: 'scWizard: A web-based automated tool for classifying and annotating single
      cells and downstream analysis of single-cell RNA-seq data in cancers'
  source_content_hash: sha256:59d42850bf159862f6870b36c170b793e8118ce3806ac936213512ca2013b392
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/Dulab2020/scWizard
  repo_url: https://github.com/Dulab2020/scWizard
  tools:
  - R
  - Seurat
  - TILPRED
  - scWizard
  - smart-seq2
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
merged_aliases:
- pca-dimensionality-reduction-of-scrnaseq
- scrna-seq-pca-dimensionality-reduction
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# pca-dimensionality-reduction-for-scrna-seq

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Dimension-reduction step of the scWizard cancer scRNA-seq workflow: PCA-based dimension reduction (as packaged in the Seurat-standardized pipeline inside scWizard) reduces the gene-by-cell counts matrix before deep-neural-network cell-type annotation, where the network's input layer is sized to the number of genes after PCA.

## When to use

Use when a raw single-cell counts matrix (csv/rds/h5, genes as rows and cells as columns) must be reduced to a lower-dimensional gene space before cell clustering or neural-network cell-type annotation in a cancer TME scRNA-seq pipeline.

## When NOT to use

- Input is already a reduced/embedded representation (e.g., pre-computed PCA or cluster assignments) rather than a raw gene-by-cell counts matrix
- Data have not passed the scWizard quality-control filters (<200 genes, >10,000 genes, >5% mitochondrial reads), since low-quality cells distort the PCA space
- Tissue-specific non-TME cell types are the target — the article notes tissue-specific cells (e.g., alveolar cells in lung cancers) may land in the Unknown cluster

## Inputs

- Single-cell RNA-seq counts matrix (H5, csv, or rds; genes as rows, cells as columns)
- Seurat object ('Data-rds' input format)
- 10X-format folder ('Data-10x' input format)
- Batch/metadata labels for Harmony or Seurat integration when multiple samples are present

## Outputs

- PCA-reduced expression representation feeding the neural-network input layer sized to the number of genes after PCA
- Dimensionally reduced object carried into Seurat cell clustering and subtype subclustering

## How to apply

In scWizard, dimension reduction is embedded in the Seurat-based standardized pipeline that also performs cell clustering and differential gene expression detection, so run it downstream of the quality-control filters (drop cells with <200 detected genes, >10,000 detected genes, or >5% mitochondrial reads) and, when multiple samples are present, after batch-effect removal with Seurat or Harmony. Apply PCA so that the reduced gene space defines the input layer of the hierarchical 'two-step' annotation model — one input layer sized to the number of genes after PCA, feeding hidden layers with ReLU and softmax activations for 7 major cell types and 47 subtypes from the 113,976-cell reference. The number of hidden nodes can be adjusted by the README criteria (e.g., between input-layer and output-layer sizes, or 2/3 of each summed), but default parameters are acceptable. The reduced representation is shared by both annotation steps: major cell-type annotation on individual cells before clustering, and subtype annotation after Seurat subclustering of each predicted major type.

## Related tools

- **scWizard** (Packages the Seurat-standardized pipeline whose dimension-reduction step produces the PCA space used to size the deep neural network's input layer for cell annotation) — https://github.com/Dulab2020/scWizard
- **Seurat** (Provides the dimension reduction (with cell clustering and differential gene expression detection) that scWizard wraps as its standardized scRNA-seq pipeline)

## Examples

```
if (!requireNamespace("devtools", quietly = TRUE)) install.packages("devtools"); devtools::install_github("Dulab2020/scWizard"); scWizard::run_app()
```

## Evaluation signals

- QC thresholds applied before PCA match scWizard defaults: cells with <200 genes, >10,000 genes, or >5% mitochondrial reads are excluded
- The annotation model's input layer dimension equals the number of genes retained after PCA
- Downstream 10-fold cross-validation on the 113,976-cell reference reaches ~0.97-0.98 concordance for major cell types and 0.79-0.89 for subtypes (myeloid ~0.85, T/NK ~0.79)
- On independent test data (e.g., the MCC dataset of 11,024 labeled cells), major cell-type annotation accuracy should be around 0.98
- Fold-level accuracy deviating strongly from the average should be flagged, per the source workflow's validation rule

## Limitations

- Tissue-specific cell types (e.g., alveolar cells in lung cancers) may not be identified and can be named Unknown
- Identification of tissue-specific subtypes and cellular states is deferred to future updates of scWizard
- Batch effects between samples should be removed (Seurat or Harmony) before annotation, otherwise the PCA/annotation space may be confounded
- The scWizard Python environment must be installed and the trainset (H5, genes as rows, cells as columns) placed in the 'app/www/python' folder before annotation

## Evidence
