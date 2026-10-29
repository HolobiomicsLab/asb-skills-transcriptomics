---
name: weighted-kde-spatial-relevance-testing
description: Use when analyzing 10X Visium spatial transcriptomics data to test whether
  curated gene sets/pathways follow anatomical or high-order spatial patterns, without
  clustering spots.
when_to_use_negative:
- Data are targeted-gene spatial panels with limited numbers of genes — GSDensity
  needs whole-transcriptomic spatial data; imputation of unmeasured genes would be
  required
- Data are dissociated single-cell (non-spatial) scRNA-seq — use the coordination/KLD
  test and RWR labeling instead of the spatial KDE relevance test
- Spot coordinates are absent or spatial grids cannot be constructed for the 2D density
  comparison
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0622
tools:
- name: GSDensity
  role: Provides compute.mca, compute.kld, compute.nn.edges, run.rwr.list, and compute.spatial.kld/compute.spatial.kld.df
    for the weighted-KDE spatial relevance test
  repo: https://github.com/qingnanl/gsdensity
- name: Seurat
  role: Loading and normalizing the 10X Visium ST count matrices before MCA embedding
- name: msigdbr
  role: Supplying curated gene sets (KEGG, BIOCARTA, Hallmarks, GO) as named lists
    for pathway scoring
- name: future.apply
  role: Parallelizing run.rwr.list pathway activity scoring across gene sets
- name: igraph
  role: Nearest-neighbor cell-gene graph used as the substrate for random-walk-with-restart
    propagation
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-023-44206-x
    title: Pathway centric analysis for single-cell RNA-seq and spatial transcriptomics
      data with GSDensity
  source_content_hash: sha256:d960eef8b617e887c771ad431015ea7b226f7f4a4f5f61a1388ca5b8b1b56e4a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/qingnanl/gsdensity
  tools:
  - GSDensity
  - Seurat
  - future.apply
  - igraph
  - msigdbr
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# weighted-kde-spatial-relevance-testing

## Summary

GSDensity's spatial-relevance test: score pathway activity per spot via random-walk-with-restart on a cell-gene nearest-neighbor graph, weight 2D cell density by these scores, and compare activity-weighted KDE against permutation background with KL divergence to call spatially relevant pathways.

## When to use

Use when analyzing 10X Visium spatial transcriptomics data to test whether curated gene sets/pathways follow anatomical or high-order spatial patterns, without clustering spots.

## When NOT to use

- Data are targeted-gene spatial panels with limited numbers of genes — GSDensity needs whole-transcriptomic spatial data; imputation of unmeasured genes would be required
- Data are dissociated single-cell (non-spatial) scRNA-seq — use the coordination/KLD test and RWR labeling instead of the spatial KDE relevance test
- Spot coordinates are absent or spatial grids cannot be constructed for the 2D density comparison

## Inputs

- Normalized 10X Visium spatial transcriptomics count matrices (Seurat objects) with spot spatial coordinates
- Curated gene set collections (e.g., MSigDB KEGG, BIOCARTA, Hallmarks, GO) as named lists
- MCA co-embedding of cells and genes (output of compute.mca)
- Per-spot pathway activity weights from run.rwr.list (restart=0.75)

## Outputs

- Per-pathway spatial-relevance p-values and adjusted p-values per dataset (compute.spatial.kld.df, n.times=100)
- Set of significant spatially relevant pathways (adjusted p < 0.05) per dataset and their cross-dataset intersection (34 pathways in the pan-cancer map)
- Activity-weighted 2D density maps / grids of pathway-relevant spots

## How to apply

First co-embed spots and genes into the shared MCA latent space with compute.mca, and pre-filter gene sets with compute.kld (n.grids=100, n.times=100 permutation nulls) so only coordinated pathways are scored. Build the spot-gene nearest-neighbor graph with compute.nn.edges(nn.use=300) and score pathway activity weights for every spot using run.rwr.list (restart=0.75, parallelized with future.apply). Convert the spatial coordinates to a gridded format and run compute.spatial.kld.df with n.times=100 permutations to test whether the activity-weighted 2D cell density deviates from the background density; pathways with adjusted p < 0.05 are called spatially relevant. Intersecting significant pathways across multiple tumor ST datasets (e.g., BC, CC, IC, OC, PACC, PC) yields recurrently spatially relevant pathways — 34 in the pan-cancer map.

## Related tools

- **GSDensity** (Provides compute.mca, compute.kld, compute.nn.edges, run.rwr.list, and compute.spatial.kld/compute.spatial.kld.df for the weighted-KDE spatial relevance test) — https://github.com/qingnanl/gsdensity
- **Seurat** (Loading and normalizing the 10X Visium ST count matrices before MCA embedding)
- **msigdbr** (Supplying curated gene sets (KEGG, BIOCARTA, Hallmarks, GO) as named lists for pathway scoring)
- **future.apply** (Parallelizing run.rwr.list pathway activity scoring across gene sets)
- **igraph** (Nearest-neighbor cell-gene graph used as the substrate for random-walk-with-restart propagation)

## Examples

```
el <- compute.nn.edges(coembed = ce, nn.use = 300); cv <- run.rwr.list(el, gene_set_list, cells, restart = 0.75, parallel = TRUE); spatial.res <- compute.spatial.kld.df(coords, activity_weights, n.times = 100)
```

## Evaluation signals

- Cross-dataset intersection of significant pathways across the six cancer ST datasets (BC, CC, IC, OC, PACC, PC) contains exactly 34 pathways at adjusted p < 0.05
- Chi-square boundary-enrichment tests for the BIOCARTA granulocyte and mesenchyme morphogenesis pathways show significant enrichment of positive-label spots inside anatomical region boundaries
- Gene-set pre-filtering via compute.kld (n.grids=100, n.times=100, gene.set.cutoff=3) marks coordinated pathways with p.adj < 0.05 before spatial scoring
- Weighted-KDE KLD statistic exceeds the permutation null distribution generated with n.times=100 spatial permutations
- Reproduced from panc.process.R in the GSDensity_manuscript_code repository

## Limitations

- GSDensity is applicable to spatial whole transcriptomic data but is not suitable for targeted gene data having limited numbers of genes; imputation of unmeasured genes could be a strategy to overcome this but is not implemented
- Neighborhood size (nn.use=300) is an important parameter when constructing the cell-gene graph
- MCA embedding is memory-intensive (~25 Gb for a 40,000-cell dataset)
- Over- or under-correction of batch effects may introduce bias, requiring careful evaluation before pathway analysis

## Evidence

- [intro] spatial relevance via activity-weighted KDE: "the spatial relevance of a pathway is reflected by the density of cells in the two-dimensional image, weighted by their pathway activity scores"
- [methods] spatial relevance functions: "Now we provide statistical test for spatial relevance of gene sets with 'compute.spatial.kld' and 'compute.spatial.kld.df' functions."
- [intro] graph propagation of pathway activity: "Pathway activity for each cell can be calculated using network propagation in a nearest-neighbor cell-gene graph, with pathway genes used as seeds for random walks"
- [discussion] treat pathway activity as cell weights: "such as what we did in treating the pathway activity as cell weights for spatial relevant pathway identification"
- [readme] spatial vignette: "The latter one is mainly focused on how to associate gene sets with the spatial information of cells or other partition of cells."
