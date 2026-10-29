---
name: concordance-validation-against-scmiko-findcdimarkers
description: Use when a re-implemented CDI-based differential expression workflow
  must be validated by reproducing the scMiko findCDIMarkers reference output — per-cluster
  top binary DE genes (bDEGs) and nCDI values in [0,1] — on an already clustered Seurat
  object.
when_to_use_negative:
- The Seurat object has no cluster assignments yet — CDI requires cluster-specific
  enrichment testing, so cluster resolution must be optimized first (e.g., via the
  marker-specificity criterion).
- The goal is standard graded differential expression across replicates — the article
  notes CDI, like other scRNA-seq DE methods, does not account for variation in biological
  replicates.
- The target is non-binary DE genes rather than bDEGs (genes exclusively expressed
  in one cell population), since CDI is designed specifically for binary differential
  expression.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scMiko
  role: R package providing getExpressedGenes for feature-space restriction and the
    reference findCDIMarkers implementation used as the concordance gold standard
  repo: https://github.com/NMikolajewicz/scMiko
- name: Seurat
  role: Framework holding the Tyser 2021 Seurat object (so.query) consumed by the
    CDI computation and validation
  repo: https://satijalab.org/seurat/
- name: scPipeline
  role: Modular Rmarkdown workflow scripts that generate the analytic reports within
    which CDI-based differential expression is run and compared
  repo: https://github.com/NMikolajewicz/scPipeline
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s42003-022-04093-2
    title: Multi-level cellular and functional annotation of single-cell transcriptomes
      using scPipeline
  source_content_hash: sha256:08f3f19b64643e1184afd74160543fcd784ed4de46c6b7f679426dee1a00902a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/NMikolajewicz/scMiko
  tools:
  - Seurat
  - scMiko
  - scPipeline
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# Concordance validation against scMiko findCDIMarkers

## Summary

A validation skill that checks a standalone implementation of the co-dependency index (CDI/nCDI) differential-expression method against the reference scMiko findCDIMarkers output on the Tyser 2021 Human Gastrulation dataset (1195 cells), confirming that per-cluster top bDEG markers and the Fig. 2j dot-plot pattern are reproduced.

## When to use

Use when a re-implemented CDI-based differential expression workflow must be validated by reproducing the scMiko findCDIMarkers reference output — per-cluster top binary DE genes (bDEGs) and nCDI values in [0,1] — on an already clustered Seurat object.

## When NOT to use

- The Seurat object has no cluster assignments yet — CDI requires cluster-specific enrichment testing, so cluster resolution must be optimized first (e.g., via the marker-specificity criterion).
- The goal is standard graded differential expression across replicates — the article notes CDI, like other scRNA-seq DE methods, does not account for variation in biological replicates.
- The target is non-binary DE genes rather than bDEGs (genes exclusively expressed in one cell population), since CDI is designed specifically for binary differential expression.

## Inputs

- Preprocessed Seurat object of the Tyser 2021 Human Gastrulation dataset (so_tyser2021_220621.rds, 1195 cells) with cluster assignments
- Reference scMiko findCDIMarkers output for the same object (for concordance comparison)
- Expressed-gene feature list from scMiko getExpressedGenes(object = so.query)

## Outputs

- Per-cluster top-marker table (cdi_markers.csv) ranking genes by nCDI
- Gene x cluster nCDI score matrix with values normalized to [0,1]
- Concordance assessment against the scMiko findCDIMarkers output and the Fig. 2j CDI Markers dot-plot pattern

## How to apply

Load the preprocessed Seurat object (so_tyser2021_220621.rds, 1195 cells) produced by the upstream preprocessing task. Restrict the feature space to genes expressed within the dataset using scMiko's getExpressedGenes(object = so.query), because CDI is computationally intensive over the full feature space. Implement the standalone CDI function: for each gene-cluster pair compute the binomial co-occurrence p-value (p_e) for enrichment of the gene's expressing cells within the cluster, set CDI = -log10(p_e), then normalize to nCDI in [0,1] per the Methods Eq. 1-4 so scores are comparable across clusters. Rank genes within each cluster by nCDI (top_n = 1, as in the vignette's cdi_output.top step) and write cdi_markers.csv. Judge success by concordance: all nCDI values must fall in [0,1], the per-cluster top markers must reproduce the scMiko findCDIMarkers output, and the markers must show the cluster-specific expression pattern seen in the CDI Markers dot plot (Fig. 2j pattern).

## Related tools

- **scMiko** (R package providing getExpressedGenes for feature-space restriction and the reference findCDIMarkers implementation used as the concordance gold standard) — https://github.com/NMikolajewicz/scMiko
- **Seurat** (Framework holding the Tyser 2021 Seurat object (so.query) consumed by the CDI computation and validation) — https://satijalab.org/seurat/
- **scPipeline** (Modular Rmarkdown workflow scripts that generate the analytic reports within which CDI-based differential expression is run and compared) — https://github.com/NMikolajewicz/scPipeline

## Examples

```
so.query <- readRDS("../data/demo/so_tyser2021_220621.rds"); expr_gene <- getExpressedGenes(object = so.query)
```

## Evaluation signals

- All computed nCDI values fall within [0,1] after the per-cluster normalization step (Methods Eq. 1-4).
- Top markers per cluster (top_n = 1 by ncdi, as in cdi_output.top) match the cluster-specific genes returned by scMiko findCDIMarkers.
- The top markers reproduce the cluster-specific bDEG pattern visible in the CDI Markers dot plot (Fig. 2j pattern).
- The feature space tested equals getExpressedGenes(object = so.query) — genes expressed within the Tyser 2021 dataset — not the full genome.
- Each per-gene CDI value equals -log10 of the binomial co-occurrence enrichment p-value (p_e) for that gene-cluster pair.

## Limitations

- CDI-based differential expression does not account for variation in biological replicates, a known limitation of existing scRNA-seq DE methods.
- CDI is computationally intensive when run over the full feature space, motivating restriction to expressed genes via getExpressedGenes.
- Cluster-dependent results inherit the stability and reproducibility limits of the clustering configuration, especially in single-replicate datasets where artifacts can produce spurious clusters.

## Evidence

- [intro] we validate the co-dependency index (CDI) as a differential expression (DE) method that identifies binary differentially-expressed genes (bDEGs): "we validate the co-dependency index (CDI) as a differential expression (DE) method that identifies binary differentially-expressed genes (bDEGs)"
- [methods] we will annotate the Human Gastrulation dataset reported by Tyser 2021: "we will annotate the Human Gastrulation dataset reported by Tyser 2021"
- [methods] so.query <- readRDS("../data/demo/so_tyser2021_220621.rds"): "so.query <- readRDS("../data/demo/so_tyser2021_220621.rds")"
- [methods] devtools::install_github(repo = "NMikolajewicz/scMiko"): "devtools::install_github(repo = "NMikolajewicz/scMiko")"
- [discussion] A known limitation of existing DE methods for scRNA-seq is the failure to account for variation in biological replicates, and the CDI approach is no exception: "A known limitation of existing DE methods for scRNA-seq is the failure to account for variation in biological replicates, and the CDI approach is no exception15."
- [readme] scMiko is an R package developed for single-cell RNA-seq analysis. It was designed using the Seurat framework: "scMiko is an **R package** developed for single-cell RNA-seq analysis. It was designed using the [Seurat](https://satijalab.org/seurat/) framework"
