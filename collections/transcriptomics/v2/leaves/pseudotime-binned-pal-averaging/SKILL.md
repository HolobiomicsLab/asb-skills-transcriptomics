---
name: pseudotime-binned-pal-averaging
description: Use when single-cell RNA-seq data has a cell pseudotime/trajectory ordering
  (e.g., monocle3) and the goal is to identify which MSigDB KEGG/BIOCARTA pathways
  are coordinated and active at successive developmental stages.
when_to_use_negative:
- Input is targeted gene-panel data with limited numbers of genes — the article states
  GSDensity 'is not suitable for targeted gene data having limited numbers of genes'
- Cells lack a meaningful trajectory/pseudotime ordering (static or spatially organized
  samples) — use spatially relevant pathway identification (compute.spatial.kld) instead
edam_operation: http://edamontology.org/operation_3929
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: GSDensity
  role: 'Core framework: compute.mca, compute.kld, compute.nn.edges, and run.rwr.list
    produce the MCA co-embedding, coordination test, cell-gene graph, and per-cell
    PALs binned along pseudotime'
  repo: https://github.com/qingnanl/gsdensity
- name: msigdbr
  role: Fetches the KEGG and BIOCARTA gene sets converted to a named list of pathway
    gene symbols
- name: cluster
  role: Provides pam for k-medoids clustering of the fitted PAL-vs-pseudotime curves
    into stage-pattern groups
- name: igraph
  role: Represents the nearest-neighbor cell-gene graph on which random walk with
    restart (label propagation) is run
- name: Seurat
  role: Data container and preprocessing (e.g., NormalizeData) for the cell-by-gene
    matrix fed to compute.mca
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-44206-x
    title: Pathway centric analysis for single-cell RNA-seq and spatial transcriptomics
      data with GSDensity
  source_content_hash: sha256:241b18bdab278a4d3b696ac8db7ae8e9e8e4364e0f836b2d08f7a75ac7efbe5a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/qingnanl/gsdensity
  tools:
  - GSDensity
  - RANN
  - Seurat
  - cluster
  - igraph
  - monocle3
  - msigdbr
  edam_operation: http://edamontology.org/operation_3929
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 14
---

# Pseudotime-binned PAL averaging

## Summary

Trajectory-coupled GSDensity analysis that averages per-cell pathway activity levels (PALs) across equal-width pseudotime bins and clusters the fitted PAL-vs-pseudotime curves, revealing curated pathways active at specific stages of development (e.g., E17.5 mouse cortex grouped into 8 k-medoids clusters).

## When to use

Use when single-cell RNA-seq data has a cell pseudotime/trajectory ordering (e.g., monocle3) and the goal is to identify which MSigDB KEGG/BIOCARTA pathways are coordinated and active at successive developmental stages.

## When NOT to use

- Input is targeted gene-panel data with limited numbers of genes — the article states GSDensity 'is not suitable for targeted gene data having limited numbers of genes'
- Cells lack a meaningful trajectory/pseudotime ordering (static or spatially organized samples) — use spatially relevant pathway identification (compute.spatial.kld) instead

## Inputs

- scRNA-seq count/normalized matrix with per-cell pseudotime ordering (e.g., monocle3 pseudotime; GSE153162 E17.5 mouse brain)
- Curated gene set collection as a named list (KEGG and BIOCARTA gene sets fetched with msigdbr)
- GSDensity coordination-test results (gene sets with p.adj < 0.05)

## Outputs

- Fitted PAL-vs-pseudotime activity curves (97 coordinated pathways × 20 pseudotime bins)
- k-medoids cluster assignment of pathway curves (8 stage-pattern groups)
- Significant pathway list with adjusted p-values from compute.kld

## How to apply

Co-embed cells and genes with compute.mca (MCA, inspired by CelliD), then run compute.kld (n.grids = 100, n.times = 100 permutations) to retain gene sets whose density in the MCA space deviates from background, using adjusted p-value < 0.05 as the coordination cutoff. Build the nearest-neighbor cell-gene graph with compute.nn.edges (nn.use = 300) and compute per-cell PALs for every coordinated pathway via run.rwr.list (random walk with restart, restart = 0.75, parallel), with pathway genes as seeds. Partition cells into 20 equal-width pseudotime bins along the monocle3 ordering and average PAL per pathway per bin to obtain fitted PAL-vs-pseudotime curves. Finally, cluster the fitted curves with k-medoids (cluster::pam) — in the E17.5 mouse cortex reference analysis, 97 KEGG/BIOCARTA pathways resolved into 8 stage-pattern clusters. Rationale: PALs reflect single-cell pathway relevance via network propagation, so binning along pseudotime converts noisy per-cell scores into smooth activity curves that expose stage-specific pathway coordination.

## Related tools

- **GSDensity** (Core framework: compute.mca, compute.kld, compute.nn.edges, and run.rwr.list produce the MCA co-embedding, coordination test, cell-gene graph, and per-cell PALs binned along pseudotime) — https://github.com/qingnanl/gsdensity
- **msigdbr** (Fetches the KEGG and BIOCARTA gene sets converted to a named list of pathway gene symbols)
- **cluster** (Provides pam for k-medoids clustering of the fitted PAL-vs-pseudotime curves into stage-pattern groups)
- **igraph** (Represents the nearest-neighbor cell-gene graph on which random walk with restart (label propagation) is run)
- **Seurat** (Data container and preprocessing (e.g., NormalizeData) for the cell-by-gene matrix fed to compute.mca)

## Examples

```
el <- compute.nn.edges(coembed = ce, nn.use = 300); pals <- run.rwr.list(el, gene_set_list, cells, restart = 0.75, parallel = TRUE)
```

## Evaluation signals

- compute.kld must yield adjusted p-value < 0.05 for the reported KEGG/BIOCARTA pathways (97 significant sets in the E17.5 mouse brain analysis)
- The 8-cluster k-medoids partition of PAL-vs-pseudotime curves must visually match the cluster composition of the published figure (Fig. 4)
- Per-gene-set PAL vectors must cover all cells returned by the graph seeds — verify every pathway gene in the named list maps into the co-embedding before run.rwr.list
- Each of the 20 pseudotime bins should contain a non-empty, roughly equal-width range of pseudotime values; empty bins invalidate the fitted curves
- Restart parameter for run.rwr.list must equal 0.75 and nn.use for compute.nn.edges must equal 300 to reproduce the reported PALs

## Limitations

- MCA embedding is memory-heavy (~25 Gb for a 40,000-cell dataset)
- Over- or under-correction of batch effects may introduce bias; evaluate batch correction before pathway analysis
- Neighborhood size (nn.use) is an important parameter when constructing the cell-gene graph; PAL binarization may yield more than two modalities and should be visually inspected

## Evidence

- [abstract] Combined with trajectory analysis, GSDensity identifies curated pathways active at various stages of mouse brain development: "GSDensity, combined with trajectory analysis can identify curated pathways that are active at various stages of mouse brain development"
- [intro] PAL computation via network propagation on nearest-neighbor cell-gene graph: "Pathway activity for each cell can be calculated using network propagation in a nearest-neighbor cell-gene graph, with pathway genes used as seeds for random walks"
- [methods] KLD coordination test parameters: "res <- compute.kld(coembed = ce, 
                   genes.use = intersect(rownames(ce), rownames(pbmc3k)),"
- [readme] gsdensity core idea and gene-set input: "Given a gene set and a cell-by-gene matrix, we want to ask: is this gene set somehow enriched by a subpopulation of the cells?"
