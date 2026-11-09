---
name: per-cluster-marker-ranking-by-ncdi
description: Use when analyzing a preprocessed Seurat object (e.g., Tyser 2021, 1195
  cells) and the goal is cluster-specific binary markers — genes expressed exclusively
  in one cell population — rather than graded average-expression DE.
when_to_use_negative:
- The goal is standard average-expression differential expression between clusters
  rather than identification of binary differentially-expressed genes (bDEGs) exclusively
  expressed in one population
- The analysis requires accounting for variation across biological replicates — CDI
  is a known exception to replicate-aware DE methods
- The gene space has not been restricted to expressed genes; running CDI on the full
  feature space is computationally intensive
edam_operation: http://edamontology.org/operation_0314
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: scMiko
  role: R package whose findCDIMarkers/getExpressedGenes workflow and vignette (cdi_output.top)
    define the CDI/nCDI computation and per-cluster top-marker ranking; installed
    via devtools::install_github
  repo: https://github.com/NMikolajewicz/scMiko
- name: Seurat
  role: Framework providing the Seurat object input (so.query) whose expression matrix
    and cluster assignments feed the binomial co-occurrence test per gene-cluster
    pair
  repo: https://satijalab.org/seurat/
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s42003-022-04093-2
    title: Multi-level cellular and functional annotation of single-cell transcriptomes
      using scPipeline
  source_content_hash: sha256:7b4a42bd29c0ed3cd2d1f2cedccff9cb589835c8d238bb74a0453625f63618e2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/NMikolajewicz/scMiko
  tools:
  - Seurat
  - scMiko
  edam_operation: http://edamontology.org/operation_0314
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# per-cluster-marker-ranking-by-ncdi

## Summary

Implements the co-dependency index (CDI) differential expression approach from scMiko/scPipeline to identify binary differentially-expressed genes (bDEGs) — genes exclusively expressed in one cell population — and ranks the top marker(s) per cluster by the normalized nCDI score.

## When to use

Use when analyzing a preprocessed Seurat object (e.g., Tyser 2021, 1195 cells) and the goal is cluster-specific binary markers — genes expressed exclusively in one cell population — rather than graded average-expression DE.

## When NOT to use

- The goal is standard average-expression differential expression between clusters rather than identification of binary differentially-expressed genes (bDEGs) exclusively expressed in one population
- The analysis requires accounting for variation across biological replicates — CDI is a known exception to replicate-aware DE methods
- The gene space has not been restricted to expressed genes; running CDI on the full feature space is computationally intensive

## Inputs

- Preprocessed Seurat object of the Tyser 2021 Human Gastrulation dataset (1195 cells), e.g. so_tyser2021_220621.rds produced by a prior preprocessing task
- Cluster identities stored in the Seurat object metadata

## Outputs

- Per-cluster top-marker table cdi_markers.csv (top_n n = 1 gene per cluster ranked by nCDI)
- Per gene-cluster nCDI matrix with values normalized to [0,1]

## How to apply

Load the preprocessed Seurat object (e.g., so_tyser2021_220621.rds) and restrict the feature space to genes expressed within the dataset using the scMiko workflow expr_gene <- getExpressedGenes(object = so.query), because CDI is computationally intensive on the full feature space. For each gene-cluster pair, compute the binomial co-occurrence p-value (p_e) for enrichment of the gene's expressing cells within the cluster (Methods Eq. 1-4) and set CDI = -log10(p_e). Normalize CDI to nCDI in [0,1] per the Methods formula so scores are comparable across clusters. Rank genes within each cluster by nCDI (top_n n = 1, as in the vignette's cdi_output.top step) and write the per-cluster top-marker table to cdi_markers.csv. Validate that all nCDI values fall within [0,1] and that top markers reproduce the cluster-specific pattern of the scMiko findCDIMarkers output and the CDI Markers dot plot (Fig. 2j pattern).

## Related tools

- **scMiko** (R package whose findCDIMarkers/getExpressedGenes workflow and vignette (cdi_output.top) define the CDI/nCDI computation and per-cluster top-marker ranking; installed via devtools::install_github) — https://github.com/NMikolajewicz/scMiko
- **Seurat** (Framework providing the Seurat object input (so.query) whose expression matrix and cluster assignments feed the binomial co-occurrence test per gene-cluster pair) — https://satijalab.org/seurat/

## Examples

```
so.query <- readRDS("../data/demo/so_tyser2021_220621.rds"); expr_gene <- getExpressedGenes(object = so.query)
```

## Evaluation signals

- All computed nCDI values fall within the [0,1] range after normalization
- Top markers per cluster reproduce the cluster-specific binary expression pattern seen in the scMiko findCDIMarkers output and the CDI Markers dot plot (Fig. 2j pattern)
- cdi_markers.csv contains exactly one top marker (top_n n = 1) per cluster, ranked by nCDI
- Feature space is restricted to genes expressed within the dataset (matching getExpressedGenes), not the full genome
- For each reported bDEG, the binomial co-occurrence p-value (p_e) shows significant enrichment of expressing cells within its assigned cluster

## Limitations

- CDI, like other existing DE methods for scRNA-seq, does not account for variation in biological replicates
- In the Tyser 2021 workflow, cluster stability/reproducibility issues in single-replicate datasets can affect which clusters the bDEG markers are assigned to

## Evidence

- [intro] CDI validates binary DE markers: "we validate the co-dependency index (CDI) as a differential expression (DE) method that identifies binary differentially-expressed genes (bDEGs)"
- [abstract] CDI-based DE is workflow step (i): "(i) co-dependency index (CDI)-based differential expression"
- [methods] Demo Seurat object input: "so.query <- readRDS("../data/demo/so_tyser2021_220621.rds")"
- [discussion] CDI limitation re biological replicates: "A known limitation of existing DE methods for scRNA-seq is the failure to account for variation in biological replicates, and the CDI approach is no exception"
- [readme] scMiko built on Seurat framework: "scMiko is an **R package** developed for single-cell RNA-seq analysis. It was designed using the [Seurat](https://satijalab.org/seurat/) framework"
