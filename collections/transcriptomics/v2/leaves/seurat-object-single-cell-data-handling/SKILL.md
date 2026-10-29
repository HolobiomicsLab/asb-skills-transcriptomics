---
name: seurat-object-single-cell-data-handling
description: Use when working with plant scRNA-seq data (e.g., Arabidopsis thaliana
  root atlas from Zhang et al., 2019) inside the scPlant pipeline and you need to
  hold expression matrices, cluster labels, GSEA results, and topic models in one
  R object.
when_to_use_negative:
- Input is already a processed feature table or non-plant scRNA-seq object outside
  the Seurat/scPlant ecosystem, since scPlant functions assume the SeuratObj contract
  and plant gene identifiers (e.g., TAIR IDs).
- Bulk RNA-seq deconvolution inputs where the starting point is a bulk expression
  matrix and cell metadata rather than per-cell scRNA-seq counts.
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_3308
tools:
- name: scPlant
  role: Wraps Seurat methods into an end-to-end pipeline where readData, QCfun, RunSeurat,
    RunGSEA_plant, and runLDA all consume/return the same SeuratObj
  repo: https://github.com/compbioNJU/scPlant
- name: Seurat
  role: Underlying data structure and methods (Seurat object) that scPlant integrates
    and extends for basic single-cell data processing
- name: CellFunTopic
  role: Consumes the SeuratObj after GSEA to visualize pathways and perform topic
    modeling; installed via devtools::install_github("compbioNJU/CellFunTopic")
  repo: https://github.com/compbioNJU/CellFunTopic
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.xplc.2023.100631
    title: 'scPlant: A versatile framework for single-cell transcriptomic data analysis
      in plants'
  source_content_hash: sha256:96a0b3772741587e2a615ae701da060feb6bbb5b0a2cd1d0a22042733850b54b
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/compbioNJU/scPlant
  repo_url: https://github.com/compbioNJU/scPlant
  tools:
  - CellFunTopic
  - Seurat
  - scPlant
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# seurat-object-single-cell-data-handling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Skill for loading, storing, and passing plant single-cell RNA-seq data through Seurat objects in the scPlant framework, where functions like readData, QCfun, RunSeurat, RunGSEA_plant, and runLDA all consume and return a single SeuratObj that carries expression matrices, clustering metadata, and enriched GO pathway signatures.

## When to use

Use when working with plant scRNA-seq data (e.g., Arabidopsis thaliana root atlas from Zhang et al., 2019) inside the scPlant pipeline and you need to hold expression matrices, cluster labels, GSEA results, and topic models in one R object.

## When NOT to use

- Input is already a processed feature table or non-plant scRNA-seq object outside the Seurat/scPlant ecosystem, since scPlant functions assume the SeuratObj contract and plant gene identifiers (e.g., TAIR IDs).
- Bulk RNA-seq deconvolution inputs where the starting point is a bulk expression matrix and cell metadata rather than per-cell scRNA-seq counts.

## Inputs

- Plant scRNA-seq expression matrix (e.g., example_Ath from the Zhang et al., 2019 Arabidopsis root dataset, downloaded from the scPlant figshare deposit 23498402)
- Seurat object produced by scPlant's readData with type='expMatrix'

## Outputs

- Seurat object (SeuratObj) enriched with QC/c clustering metadata after RunSeurat(resolution = 0.3)
- Seurat object carrying per-cluster GO GSEA signatures after RunGSEA_plant
- Seurat object with LDA topic-model results (k = 23) after runLDA, extractable via tidytext::tidy

## How to apply

Load raw count data into a Seurat object with scPlant's readData (type='expMatrix'), then run QC and clustering via QCfun and RunSeurat (e.g., resolution = 0.3) so that cluster identities are stored in the object's metadata. Every downstream scPlant module — automatic cell-type annotation, RunGSEA_plant, runLDA — follows the same contract: it takes the SeuratObj as input and returns the same SeuratObj with new assays/metadata attached, so you never work outside the object. For functional annotation, call RunGSEA_plant(SeuratObj, by='GO', GeneIDtype='TAIR', minpct=0.25) to attach enriched GO pathway signatures per cluster, and runLDA(SeuratObj, k=23) to attach topic-model results for 23 clusters. The rationale is that scPlant integrates Seurat methods directly ('by integrating methods of Seurat package'), so the Seurat object is the single stateful container threading QC, clustering, GSEA scoring, and CellFunTopic topic modeling together.

## Related tools

- **scPlant** (Wraps Seurat methods into an end-to-end pipeline where readData, QCfun, RunSeurat, RunGSEA_plant, and runLDA all consume/return the same SeuratObj) — https://github.com/compbioNJU/scPlant
- **Seurat** (Underlying data structure and methods (Seurat object) that scPlant integrates and extends for basic single-cell data processing)
- **CellFunTopic** (Consumes the SeuratObj after GSEA to visualize pathways and perform topic modeling; installed via devtools::install_github("compbioNJU/CellFunTopic")) — https://github.com/compbioNJU/CellFunTopic

## Examples

```
SeuratObj <- RunSeurat(SeuratObj, resolution = 0.3)
```

## Evaluation signals

- The workflow code pattern holds: every step reassigns the same variable (e.g., SeuratObj <- QCfun(SeuratObj); SeuratObj <- RunSeurat(SeuratObj, resolution = 0.3)), confirming the object round-trips through each function.
- After RunGSEA_plant, each cluster must carry enriched GO pathway signatures computed with GeneIDtype='TAIR' and minpct=0.25.
- runLDA must yield exactly k topics matching the number of clusters (k = 23 for the Arabidopsis root data), with beta and gamma matrices retrievable via tidytext::tidy.
- The functional-similarity network built from the SeuratObj-derived topic/pathway profiles should place the unannotated cluster 18 adjacent to cortex clusters, matching Figure 3F of the paper.
- The data must originate from a plant knowledgebase-supported genome (e.g., Arabidopsis TAIR10) so that GO-based gene ID mapping succeeds.

## Limitations

- The SeuratObj contract assumes plant species with supported genome/annotation resources (TAIR10, Zm-B73-REFERENCE-NAM-5.0, Oryza sativa V7.0); other organisms require downloading genome and gff3 files from EnsemblPlants or Phytozome.
- scPlant is a plant-specific framework — it is not intended as a general-purpose scRNA-seq solution for non-plant data, since the underlying knowledgebase is plant-oriented.
- k for runLDA must match the number of clusters (e.g., 23); mismatched k will not correspond to cluster-level topic profiles.

## Evidence
