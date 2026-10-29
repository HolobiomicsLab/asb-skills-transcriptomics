---
name: single-cell-cluster-pooling-and-preparation
description: Use when scRNA-seq data stored as a Seurat object must be converted into
  pooled, cluster-level transcriptomic profiles (rows = genes, columns = clusters)
  before predicting miRNA expression with miRSCAPE or a similar trained model.
when_to_use_negative:
- The input is already a dense bulk RNA-seq or miRNA expression matrix and only needs
  bulkTransform-style log normalization, not Seurat-object pooling
- The task requires single-cell-resolution miRNA inference without cluster pooling
  (e.g. following miReact, which interprets predicted miRNA at the level of cell clusters
  anyway, pooling is the intended resolution)
- The scRNA-seq data is not in a Seurat object and cannot be converted, since modifySeuratObject
  assumes Seurat object storage
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: miRSCAPE
  role: Target pipeline whose miRSCAPE function consumes the pooled scmRNA output;
    also provides modifySeuratObject and bulkTransform for preparation
  repo: https://github.com/hannenhalli-lab/miRSCAPE
- name: Seurat
  role: R library required to load and manipulate the Seurat object before pooling
    cells within clusters
- name: xgboost
  role: Dependency backing the miRSCAPE prediction model applied to the pooled profiles
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.isci.2022.104962
    title: miRSCAPE - inferring miRNA expression from scRNA-seq data
  source_content_hash: sha256:2df34f7ddd287daef02a51d478c5e71d57ce523b6268fcf5f9260c29ceaa08cf
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
  - Seurat
  - miRSCAPE
  - xgboost
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# single-cell-cluster-pooling-and-preparation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Prepares Seurat-object scRNA-seq data for miRNA-expression inference by pooling cells within each cell-type cluster and formatting the pooled transcriptomic matrix, so the sparse single-cell profile becomes a dense sample-level input for the miRSCAPE xgboost model. It also covers bulkData-style transformation (bulkTransform) of the accompanying mRNA and miRNA matrices.

## When to use

Use when scRNA-seq data stored as a Seurat object must be converted into pooled, cluster-level transcriptomic profiles (rows = genes, columns = clusters) before predicting miRNA expression with miRSCAPE or a similar trained model.

## When NOT to use

- The input is already a dense bulk RNA-seq or miRNA expression matrix and only needs bulkTransform-style log normalization, not Seurat-object pooling
- The task requires single-cell-resolution miRNA inference without cluster pooling (e.g. following miReact, which interprets predicted miRNA at the level of cell clusters anyway, pooling is the intended resolution)
- The scRNA-seq data is not in a Seurat object and cannot be converted, since modifySeuratObject assumes Seurat object storage

## Inputs

- scRNA-seq data stored as a Seurat object (genes as rows, cells as columns, gene names in row names)
- Cell-type cluster identifiers to select clusters of interest (e.g. 'Ductal cell type 1')
- Paired bulk mRNA expression matrix for transformation (e.g. pdac_mrna.txt)
- Paired bulk miRNA expression matrix for log-only transformation (e.g. pdac_mirna.txt)

## Outputs

- Pooled per-cluster transcriptomic matrix (scmRNA) suitable for miRSCAPE prediction
- Transformed bulk mRNA matrix (bulkk_mrna) from bulkTransform
- Log-transformed, unscaled bulk miRNA matrix (bulkk_mirna) from bulkTransform(justNorm = TRUE)
- Predicted miRNA expression values returned in pred, enabling downstream auROC evaluation

## How to apply

Load the Seurat library and the miRSCAPE.R source in R 4.1.0, then decide whether to prepare all clusters or only specific cell types: passing clusterId (e.g. clustId = c('Ductal cell type 1')) restricts preparation to those clusters, while omitting it prepares all clusters. Use modifySeuratObject to extract and pool cells within each cluster, since pooling collapses sparse single-cell profiles into dense per-cluster profiles that miRSCAPE can consume; rows are genes and columns are samples/clusters, and gene names must be supplied as row names. For paired bulk mRNA/miRNA inputs, apply bulkTransform to the mRNA data (default scaling) and call bulkTransform with justNorm = TRUE on the miRNA data to log-transform without scaling. Feed the prepared objects as bulkmRNA, bulkmiRNA, and scmRNA arguments to the miRSCAPE function, whose predicted expressions are returned in pred. When validating against detected-versus-undetected miRNAs (e.g. auROC ~0.71 for ENCODE cell lines, ~0.67 for mouse hematopoietic cell types), pool cells within each cell type cluster before applying the pre-trained AML-TCGA-trained model.

## Related tools

- **miRSCAPE** (Target pipeline whose miRSCAPE function consumes the pooled scmRNA output; also provides modifySeuratObject and bulkTransform for preparation) — https://github.com/hannenhalli-lab/miRSCAPE
- **Seurat** (R library required to load and manipulate the Seurat object before pooling cells within clusters)
- **xgboost** (Dependency backing the miRSCAPE prediction model applied to the pooled profiles)

## Examples

```
clustId = c('Ductal cell type 1'); denem = modifySeuratObject(pbmc = example, clusterId = clustId)
```

## Evaluation signals

- Pooled matrix has gene names as row names and one column per cluster/patient (rows are genes, columns are sample values)
- miRNA data passed through bulkTransform with justNorm = TRUE is log-transformed but NOT scaled
- miRSCAPE prediction returns non-empty predicted expressions in pred for the selected clusters
- Downstream auROC of predicted expression for detected vs undetected miRNAs falls in the reported reasonable-accuracy range (~0.67 for mouse hematopoietic cell types trained on AML-TCGA; ~0.71 for ENCODE cell lines)
- Restricted preparation with clusterId yields profiles only for the specified clusters (e.g. 'Ductal cell type 1'), while omitting clusterId covers all clusters

## Limitations

- Given the large dropout rate in scRNA-seq, a pre-trained model may not work and miRSCAPE may need to be retrained using only the genes detected at a reasonable level
- Model accuracy is expected to be slightly reduced when the cellular/tissue context of the application sample does not match the training cohort (e.g. CCLE- or AML-TCGA-trained models), with more substantive loss when training and application contexts differ substantially
- miRSCAPE needs to be trained on a large cohort of paired miRNA-mRNA data, unlike prior target-based approaches that require no specific training

## Evidence
