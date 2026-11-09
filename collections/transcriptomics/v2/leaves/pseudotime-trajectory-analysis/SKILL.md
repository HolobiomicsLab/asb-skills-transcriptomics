---
name: pseudotime-trajectory-analysis
description: Use when scRNA-seq cells have a pseudotime/trajectory ordering (e.g.
  monocle3 on GSE153162 E17.5 mouse cortex) and the goal is to find which pathway
  gene sets are active at each developmental stage and how their activity curves cluster
  over pseudotime.
when_to_use_negative:
- Data are spatial transcriptomics with no trajectory ordering; use the weighted-KDE
  spatial relevance workflow (compute.spatial.kld) instead of pseudotime binning
- Input is targeted gene-panel data with limited numbers of genes; GSDensity is not
  suitable for such data and imputation is suggested but not implemented
- Only pathway-level enrichment of a predefined cluster is needed without developmental
  ordering; a simpler coordination test (compute.kld alone) suffices
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: GSDensity
  role: 'Core framework: MCA co-embedding, KLD coordination test, and RWR-based per-cell
    PAL computation along pseudotime'
  repo: https://github.com/qingnanl/gsdensity
- name: monocle3
  role: Provides the pseudotime trajectory ordering of E17.5 mouse brain cells used
    to bin cells and fit PAL curves
- name: msigdbr
  role: Fetches KEGG and BIOCARTA gene sets from MSigDB and converts them to a named
    gene-symbol list
- name: cluster (pam)
  role: K-medoids clustering of the 97 fitted PAL-vs-pseudotime curves into 8 stage-pattern
    groups
- name: igraph / RANN
  role: Nearest-neighbor cell-gene graph construction (compute.nn.edges, nn.use =
    300) and random walk with restart propagation
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-44206-x
    title: Pathway centric analysis for single-cell RNA-seq and spatial transcriptomics
      data with GSDensity
  source_content_hash: sha256:1e8c57e7e7b34ff2949a1f1c65d6b3a5d87eb9ea28b36cf4197796e3fb6b5455
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/qingnanl/gsdensity
  tools:
  - GSDensity
  - RANN
  - Seurat
  - cluster (pam)
  - igraph
  - igraph / RANN
  - monocle3
  - msigdbr
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8666666666666667
n_claims: 15
---

# pseudotime-trajectory-analysis

## Summary

Combine monocle3 pseudotime ordering of single-cell data with GSDensity pathway activity levels (PALs) to identify curated KEGG/BIOCARTA pathways active at successive stages of mouse brain development, grouping fitted PAL-vs-pseudotime curves into stage-pattern clusters via k-medoids.

## When to use

Use when scRNA-seq cells have a pseudotime/trajectory ordering (e.g. monocle3 on GSE153162 E17.5 mouse cortex) and the goal is to find which pathway gene sets are active at each developmental stage and how their activity curves cluster over pseudotime.

## When NOT to use

- Data are spatial transcriptomics with no trajectory ordering; use the weighted-KDE spatial relevance workflow (compute.spatial.kld) instead of pseudotime binning
- Input is targeted gene-panel data with limited numbers of genes; GSDensity is not suitable for such data and imputation is suggested but not implemented
- Only pathway-level enrichment of a predefined cluster is needed without developmental ordering; a simpler coordination test (compute.kld alone) suffices

## Inputs

- E17.5 mouse brain scRNA-seq count matrix (GEO GSE153162)
- monocle3 pseudotime ordering per cell
- KEGG and BIOCARTA gene sets from MSigDB via msigdbr (as a named list of gene symbols)

## Outputs

- 97 significantly coordinated pathways with adjusted p-value < 0.05
- Per-cell PAL (pathway activity level) matrix from random walk with restart
- Fitted PAL-vs-pseudotime curves per pathway (20 equal-width pseudotime bins)
- 8-cluster k-medoids (pam) partition of pathway activity curves corresponding to developmental stage patterns

## How to apply

Load the E17.5 mouse brain scRNA-seq data (GSE153162) together with the monocle3 pseudotime value per cell, and fetch KEGG and BIOCARTA gene sets from MSigDB with msigdbr as a named list. Co-embed cells and genes with compute.mca (MCA), then run compute.kld (n.grids = 100, n.times = 100 permutations) and retain gene sets with adjusted p-value < 0.05 as significantly coordinated pathways (97 KEGG/BIOCARTA sets reported). Build the cell-gene nearest-neighbor graph with compute.nn.edges (nn.use = 300) and compute per-cell PALs for all coordinated pathways using run.rwr.list with restart = 0.75 in parallel. Partition cells into 20 equal-width pseudotime bins and average PALs per pathway per bin to obtain fitted PAL-vs-pseudotime curves. Finally, cluster the 97 fitted curves into 8 groups with k-medoids (cluster::pam) and compare the cluster composition against the published figure. Rationale: MCA co-embedding lets pathway genes act as seeds for random walks with restart on the cell-gene graph, so pathway activity becomes a per-cell quantity that can be tracked along pseudotime.

## Related tools

- **GSDensity** (Core framework: MCA co-embedding, KLD coordination test, and RWR-based per-cell PAL computation along pseudotime) — https://github.com/qingnanl/gsdensity
- **monocle3** (Provides the pseudotime trajectory ordering of E17.5 mouse brain cells used to bin cells and fit PAL curves)
- **msigdbr** (Fetches KEGG and BIOCARTA gene sets from MSigDB and converts them to a named gene-symbol list)
- **cluster (pam)** (K-medoids clustering of the 97 fitted PAL-vs-pseudotime curves into 8 stage-pattern groups)
- **igraph / RANN** (Nearest-neighbor cell-gene graph construction (compute.nn.edges, nn.use = 300) and random walk with restart propagation)

## Examples

```
ce <- compute.mca(object = seurat_obj); el <- compute.nn.edges(coembed = ce, nn.use = 300); pals <- run.rwr.list(el, gene_set_list, cells, restart = 0.75, parallel = TRUE)
```

## Evaluation signals

- The compute.kld coordination test must yield adjusted p-value < 0.05 for the 97 reported KEGG/BIOCARTA pathways
- The k-medoids partition must produce 8 clusters whose composition visually matches the published Fig. 4 stage-pattern clusters
- Per-cell PALs must be computed on the graph built with nn.use = 300 and restart = 0.75, run in parallel over all coordinated gene sets
- Binned curves must use 20 equal-width pseudotime bins with averaged PALs per pathway per bin
- MCA embedding memory footprint should be planned (~25 Gb for a 40,000-cell dataset); verify the pipeline completes without memory exhaustion

## Limitations

- GSDensity is not suitable for targeted gene data with limited numbers of genes; imputation of unmeasured genes is suggested but not implemented
- Neighborhood size (nn.use) is an important parameter when constructing the cell-gene graph and affects results
- MCA embedding requires large memory (~25 Gb for a 40,000-cell dataset); a linux server with parallel computing is recommended when testing many gene sets

## Evidence

- [abstract] GSDensity, combined with trajectory analysis can identify curated pathways that are active at various stages of mouse brain development: "GSDensity, combined with trajectory analysis can identify curated pathways that are active at various stages of mouse brain development"
- [intro] RWR seeds on cell-gene graph: "Pathway activity for each cell can be calculated using network propagation in a nearest-neighbor cell-gene graph, with pathway genes used as seeds for random walks"
- [methods] nn.edges parameter: "el <- compute.nn.edges(coembed = ce, nn.use = 300)"
- [methods] run.rwr.list signature: "run.rwr.list(el, gene_set_list, cells, restart, parallel)"
- [readme] manuscript code covers the mouse brain development application: "mb.r: mouse brain spatial

mouse_brain_dev.r: mouse brain development"
