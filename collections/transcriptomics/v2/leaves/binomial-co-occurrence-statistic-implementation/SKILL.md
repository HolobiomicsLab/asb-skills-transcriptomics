---
name: binomial-co-occurrence-statistic-implementation
description: Use when hunting for genes expressed almost exclusively in one cell cluster
  (bDEGs) in a scRNA-seq Seurat object, and when per-cluster marker scores must be
  normalized to [0,1] so they are comparable across clusters.
when_to_use_negative:
- The dataset has multiple biological replicates whose variation must be modeled —
  the article states the CDI approach does not account for variation in biological
  replicates
- The goal is ranking genes by average/continuous expression differences rather than
  identifying genes expressed exclusively (binary on/off pattern) in one cluster
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: scMiko
  role: R package providing getExpressedGenes for feature-space restriction and findCDIMarkers,
    whose output serves as the reference for validating the standalone CDI/nCDI implementation
  repo: https://github.com/NMikolajewicz/scMiko
- name: Seurat
  role: Framework holding the preprocessed scRNA-seq object (counts, clusters, metadata)
    on which the CDI computation is run
- name: scPipeline
  role: Modular Rmarkdown workflow whose CDI-based differential expression module
    defines the CDI/nCDI steps this skill re-implements standalone
  repo: https://github.com/NMikolajewicz/scPipeline
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s42003-022-04093-2
    title: Multi-level cellular and functional annotation of single-cell transcriptomes
      using scPipeline
  source_content_hash: sha256:a1624046c783b381dd906cbdbf0625250afa95beedc31a44e5d25e2a95fa3dc9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/NMikolajewicz/scMiko
  tools:
  - Seurat
  - scMiko
  - scPipeline
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# binomial-co-occurrence-statistic-implementation

## Summary

Implements the co-dependency index (CDI)/nCDI differential expression statistic from scPipeline/scMiko, which scores each gene-cluster pair by a binomial co-occurrence p-value to identify cluster-specific binary differentially-expressed genes (bDEGs) in scRNA-seq data.

## When to use

Use when hunting for genes expressed almost exclusively in one cell cluster (bDEGs) in a scRNA-seq Seurat object, and when per-cluster marker scores must be normalized to [0,1] so they are comparable across clusters.

## When NOT to use

- The dataset has multiple biological replicates whose variation must be modeled — the article states the CDI approach does not account for variation in biological replicates
- The goal is ranking genes by average/continuous expression differences rather than identifying genes expressed exclusively (binary on/off pattern) in one cluster

## Inputs

- Preprocessed Seurat object of expressed cells (e.g. so_tyser2021_220621.rds, 1195 cells)
- List of genes expressed within the dataset (from getExpressedGenes)
- Cluster assignments in the Seurat object metadata (e.g. seurat_clusters)

## Outputs

- Per-cluster top-marker table written to cdi_markers.csv
- Gene x cluster matrix of CDI (-log10 binomial co-occurrence p-value) values
- Gene x cluster matrix of nCDI values normalized to [0,1]

## How to apply

Load the preprocessed Seurat object (e.g. so_tyser2021_220621.rds, 1195 cells from the Tyser 2021 Human Gastrulation dataset produced by an upstream preprocessing task). Restrict the feature space to genes expressed in the dataset using scMiko's getExpressedGenes(object = so.query), since CDI is computationally intensive over the full feature space. For each gene-cluster pair, compute the binomial co-occurrence p-value (p_e) for enrichment of the gene's expressing cells within the cluster and set CDI = -log10(p_e), per Methods Eq. 1-4. Normalize CDI to nCDI in [0,1] so scores are comparable across clusters, then rank genes within each cluster by nCDI (top_n = 1 by ncdi, as in the vignette's cdi_output.top step) and export the per-cluster top-marker table to cdi_markers.csv.

## Related tools

- **scMiko** (R package providing getExpressedGenes for feature-space restriction and findCDIMarkers, whose output serves as the reference for validating the standalone CDI/nCDI implementation) — https://github.com/NMikolajewicz/scMiko
- **Seurat** (Framework holding the preprocessed scRNA-seq object (counts, clusters, metadata) on which the CDI computation is run)
- **scPipeline** (Modular Rmarkdown workflow whose CDI-based differential expression module defines the CDI/nCDI steps this skill re-implements standalone) — https://github.com/NMikolajewicz/scPipeline

## Examples

```
expr_gene <- getExpressedGenes(object = so.query); cdi_output <- findCDIMarkers(object = so.query, features = expr_gene, top_n = 1)
```

## Evaluation signals

- All computed nCDI values fall within [0,1] per the Methods normalization formula
- Per-cluster top markers (top_n = 1 by ncdi) reproduce the cluster-specific pattern of the scMiko findCDIMarkers output
- CDI Markers dot plot reproduces the Fig. 2j pattern of cluster-restricted marker expression
- Top markers per cluster show a binary expressing-cell pattern (expressing cells enriched within the assigned cluster under the binomial co-occurrence test)
- Feature space matches getExpressedGenes output (genes expressed in the dataset), not the full genome annotation

## Limitations

- A known limitation of existing DE methods for scRNA-seq is the failure to account for variation in biological replicates, and the CDI approach is no exception
- CDI is computationally intensive on the full feature space, so analysis must be restricted to genes expressed within the dataset
- Binary bDEG scores depend on the cluster configuration; cluster stability in single-replicate datasets limits reproducibility of the resulting markers

## Evidence

- [intro] CDI validates as binary DE method: "we validate the co-dependency index (CDI) as a differential expression (DE) method that identifies binary differentially-expressed genes (bDEGs)"
- [abstract] CDI-based DE workflow module: "(i) co-dependency index (CDI)-based differential expression"
- [discussion] CDI replicate limitation: "A known limitation of existing DE methods for scRNA-seq is the failure to account for variation in biological replicates, and the CDI approach is no exception"
