---
name: bootstrapped-cell-type-profile-sampling
description: Use when scRNA-seq data stored in a Seurat object must be converted into
  non-sparse per-cluster pseudo-bulk mRNA profiles (rows = genes, gene names as row
  names) before miRNA prediction with miRSCAPE.
when_to_use_negative:
- Input is already bulk RNA-seq data — use bulkTransform instead of modifySeuratObject
- Per-single-cell miRNA inference is required — the article states miReact's single-cell
  predictions were interpreted at cluster level and miRSCAPE pools cells within a
  cluster
- Genes are largely undetected due to dropout such that a pre-trained model will not
  work — retraining on detected genes may be needed rather than plain pooling
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: miRSCAPE
  role: Provides modifySeuratObject, the function that pools/bootstraps cells within
    a Seurat cluster into a non-sparse profile for miRNA prediction
  repo: https://github.com/hannenhalli-lab/miRSCAPE
- name: Seurat
  role: Required dependency; scRNA-seq data must be stored in a Seurat object before
    modifySeuratObject is applied
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.isci.2022.104962
    title: miRSCAPE - inferring miRNA expression from scRNA-seq data
  source_content_hash: sha256:3c1605fda1e98aa03a80e20ac71db14f39dbb1d1c36ba15554fd460ead74a08e
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
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# bootstrapped-cell-type-profile-sampling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Prepare scRNA-seq profiles for downstream miRNA inference by pooling (bootstrapped sampling of) cells within each Seurat cluster to produce non-sparse pseudo-bulk mRNA profiles, using the miRSCAPE modifySeuratObject function.

## When to use

Use when scRNA-seq data stored in a Seurat object must be converted into non-sparse per-cluster pseudo-bulk mRNA profiles (rows = genes, gene names as row names) before miRNA prediction with miRSCAPE.

## When NOT to use

- Input is already bulk RNA-seq data — use bulkTransform instead of modifySeuratObject
- Per-single-cell miRNA inference is required — the article states miReact's single-cell predictions were interpreted at cluster level and miRSCAPE pools cells within a cluster
- Genes are largely undetected due to dropout such that a pre-trained model will not work — retraining on detected genes may be needed rather than plain pooling

## Inputs

- Seurat object containing single-cell mRNA profiles (example data stored as .rds, e.g. example = readRDS('example data/example.rds'))
- Cell cluster identifiers to select specific cell types (e.g. clustId = c('Ductal cell type 1')); omit for all clusters

## Outputs

- Non-sparse pooled (pseudo-bulk) scRNA-seq mRNA matrix per cell cluster, rows = genes with gene names as row names, suitable as the scmRNA input to the miRSCAPE function

## How to apply

Load the Seurat object and source miRSCAPE.R (R 4.1.0, with xgboost and Seurat installed). Run modifySeuratObject on the Seurat object, passing the clusters of interest via the clusterId parameter (e.g. clustId = c('Ductal cell type 1')) or omitting clusterId to prepare all cell clusters. The function pools cells within a single cell cluster — bootstrapped sampling of cells — so the resulting transcriptomic profile is pooled and therefore not sparse, addressing scRNA-seq dropout. The output is a pooled scRNA-seq matrix (rows are genes, gene names as row names) that can be passed directly as the scmRNA argument to the miRSCAPE prediction function. Rationale: mature miRNAs cannot be profiled per-cell because standard scRNA-seq protocols rely on poly(A) capture, so miRNA activity is inferred at the cluster level from the pooled mRNA profile.

## Related tools

- **miRSCAPE** (Provides modifySeuratObject, the function that pools/bootstraps cells within a Seurat cluster into a non-sparse profile for miRNA prediction) — https://github.com/hannenhalli-lab/miRSCAPE
- **Seurat** (Required dependency; scRNA-seq data must be stored in a Seurat object before modifySeuratObject is applied)

## Examples

```
clustId = c('Ductal cell type 1'); denem = modifySeuratObject(pbmc = example, clusterId = clustId)
```

## Evaluation signals

- The pooled output matrix is non-sparse (dropout reduced) relative to the raw per-cell expression matrix
- Row names of the output are valid gene names, matching the requirement that 'Rows are genes, columns are patient values' with gene names as row names
- Only cells from the clusters specified via clusterId are included when clusterId is supplied; all clusters are included when it is omitted
- Downstream miRSCAPE run on the pooled profile returns predictions retrievable from the pred output slot

## Limitations

- Given the large dropout rate in scRNA-seq, a pre-trained model may not work on the pooled profile and miRSCAPE may need to be retrained using only genes detected at a reasonable level
- Model accuracy is reduced when the cellular/tissue context of the application sample does not match the training cohort
- Pooling restricts inference to cell-cluster level; single-cell-resolution miRNA expression is not recovered

## Evidence

- [discussion] we pool cells within a single cell cluster and apply miRSCAPE to the pooled (and therefore not sparse) transcriptomic profile to infer miRNA activity: "we pool cells within a single cell cluster and apply miRSCAPE to the pooled (and therefore not sparse) transcriptomic profile to infer miRNA activity"
- [readme] To prepare the data which is stored in Seurat object, `modifySeuratObject` function should be used.: "To prepare the data which is stored in Seurat object, `modifySeuratObject` function should be used."
- [readme] denem = modifySeuratObject(pbmc = example, clusterId = clustId): "denem = modifySeuratObject(pbmc = example, clusterId = clustId)"
- [readme] Rows are genes, columns are patient values. In order to properly run the code, gene names as row names must be provided.: "Rows are genes, columns are patient values. In order to properly run the code, gene names as row names must be provided."
