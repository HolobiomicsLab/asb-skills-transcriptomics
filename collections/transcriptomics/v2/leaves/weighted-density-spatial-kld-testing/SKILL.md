---
name: weighted-density-spatial-kld-testing
description: Use when spatial transcriptomics data with spatial coordinates is available
  and you must decide whether a gene set/pathway shows spatial organization beyond
  chance, rather than just cell-type enrichment.
when_to_use_negative:
- Targeted (panel-based) spatial data with limited numbers of genes — the paper states
  GSDensity 'is not suitable for targeted gene data having limited numbers of genes'
- Data lacking spatial coordinates — the weighted-density test requires a two-dimensional
  spatial map of cells
- Pathway activity without coordination testing — the article recommends binarization/interpretation
  only on gene sets that pass the coordination test
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: GSDensity
  role: Provides compute.mca, compute.nn.edges, run.rwr/run.rwr.list, compute.spatial.kld
    and compute.spatial.kld.df for activity-weighted spatial KLD testing
  repo: https://github.com/qingnanl/gsdensity
- name: CelliD
  role: 'Inspiration/benchmark: its MCA-based cell ranking is used as the parallel
    method for computing pathway activity in the spatial relevance benchmark'
  repo: https://bioconductor.org/packages/devel/bioc/html/CelliD.html
- name: SRTsim
  role: Simulates the four ST spatial patterns (hotspot, hotspot-with-gradient, streak,
    gradient) with embedded ground-truth and random gene sets for validation
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-023-44206-x
    title: Pathway centric analysis for single-cell RNA-seq and spatial transcriptomics
      data with GSDensity
  source_content_hash: sha256:00a271fc642b4c70e4bcbb38f87154711397a456528d74717d4459288c9415a6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/qingnanl/gsdensity
  tools:
  - CelliD
  - GSDensity
  - SRTsim
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# Weighted-density spatial KLD testing

## Summary

A GSDensity procedure that tests whether a gene set is spatially relevant by weighting the 2D cell-density map with per-cell pathway activity levels (PAL scores from random-walk-with-rest propagation) and comparing Kullback-Leibler divergence against permuted nulls.

## When to use

Use when spatial transcriptomics data with spatial coordinates is available and you must decide whether a gene set/pathway shows spatial organization beyond chance, rather than just cell-type enrichment.

## When NOT to use

- Targeted (panel-based) spatial data with limited numbers of genes — the paper states GSDensity 'is not suitable for targeted gene data having limited numbers of genes'
- Data lacking spatial coordinates — the weighted-density test requires a two-dimensional spatial map of cells
- Pathway activity without coordination testing — the article recommends binarization/interpretation only on gene sets that pass the coordination test

## Inputs

- Spatial transcriptomics count matrix with per-cell 2D spatial coordinates (e.g., 10x Visium or SRTsim-simulated modes: hotspot, hotspot-with-gradient, streak, gradient)
- MCA co-embedding of cells and genes (from compute.mca)
- Nearest-neighbor cell-gene edge list (from compute.nn.edges, nn.use = 300)
- Gene sets of interest as a named list (e.g., MSigDB GO:BP, Hallmarks, KEGG, BIOCARTA)

## Outputs

- Per-cell PAL (pathway activity level) scores for each gene set
- Per-gene-set spatial KLD statistics and permutation p-values (from compute.spatial.kld / compute.spatial.kld.df)
- Adjusted p-values flagging spatially relevant gene sets (p.adj < 0.05)
- Activity-weighted 2D density maps per pathway

## How to apply

First co-embed cells and genes with compute.mca (MCA, the CelliD-style embedding) and build a nearest-neighbor cell-gene graph with compute.nn.edges using nn.use = 300. Compute per-cell pathway activity levels for each gene set via run.rwr / run.rwr.list with the default restart = 0.75. Then apply compute.spatial.kld or compute.spatial.kld.df to the 2D cell density weighted by these PAL scores, using n.times label-permutation iterations to obtain a KLD statistic and empirical p-value for spatial relevance. Call a gene set spatially relevant when the activity-weighted density yields significantly higher KLD than permuted controls (adjusted p < 0.05). Rationale: spatial relevance is reflected by the density of cells in the two-dimensional image weighted by pathway activity, so a pathway concentrated in spatial niches shows a weighted density diverging strongly from the unweighted/permuted null.

## Related tools

- **GSDensity** (Provides compute.mca, compute.nn.edges, run.rwr/run.rwr.list, compute.spatial.kld and compute.spatial.kld.df for activity-weighted spatial KLD testing) — https://github.com/qingnanl/gsdensity
- **CelliD** (Inspiration/benchmark: its MCA-based cell ranking is used as the parallel method for computing pathway activity in the spatial relevance benchmark) — https://bioconductor.org/packages/devel/bioc/html/CelliD.html
- **SRTsim** (Simulates the four ST spatial patterns (hotspot, hotspot-with-gradient, streak, gradient) with embedded ground-truth and random gene sets for validation)

## Examples

```
cv <- run.rwr.list(el, gene_set_list, cells, restart = 0.75); kld_res <- compute.spatial.kld.df(cell_df = weighted_cell_density, n.times = 100)
```

## Evaluation signals

- In SRTsim simulations of all four spatial modes (hotspot, hotspot-with-gradient, streak, gradient), ground-truth spatially relevant gene sets show significantly higher spatial KLD than matched random gene sets (adjusted p < 0.05)
- Permutation null behaves as expected: KLD p-values from n.times label permutations are uniformly distributed for non-spatial (random) gene sets
- PAL scores from run.rwr use restart = 0.75 on the nn.use = 300 cell-gene graph and produce a full per-cell probability vector aligned to the spatial coordinates
- Benchmarking with CelliD's MCA-based ranking as an alternative activity scorer still separates ground-truth from random gene sets, confirming the weighting step (not the scorer) drives the KLD separation
- Results table assembled per simulation mode reports delta-KLD (ground-truth minus random separation) with positive values in every mode

## Limitations

- GSDensity is applicable to spatial whole transcriptomic data but not suitable for targeted gene data with limited numbers of genes; imputation of unmeasured genes is suggested but not implemented
- Neighborhood size is an important parameter when constructing the cell-gene graph (nn.use), and results can depend on this choice
- MCA embedding is memory-intensive (~25 Gb for a 40,000-cell dataset)
- Proper batch correction is needed; over- or under-correction may introduce bias into pathway analysis

## Evidence

- [intro] the spatial relevance of a pathway is reflected by the density of cells in the two-dimensional image, weighted by their pathway activity scores: "the spatial relevance of a pathway is reflected by the density of cells in the two-dimensional image, weighted by their pathway activity scores"
- [intro] Pathway activity for each cell can be calculated using network propagation in a nearest-neighbor cell-gene graph, with pathway genes used as seeds for random walks: "Pathway activity for each cell can be calculated using network propagation in a nearest-neighbor cell-gene graph, with pathway genes used as seeds for random walks"
- [methods] Now we provide statistical test for spatial relevance of gene sets with 'compute.spatial.kld' and 'compute.spatial.kld.df' functions.: "Now we provide statistical test for spatial relevance of gene sets with 'compute.spatial.kld' and 'compute.spatial.kld.df' functions."
- [discussion] such as what we did in treating the pathway activity as cell weights for spatial relevant pathway identification: "such as what we did in treating the pathway activity as cell weights for spatial relevant pathway identification"
- [readme] el <- compute.nn.edges(coembed = ce, nn.use = 300): "el <- compute.nn.edges(coembed = ce, nn.use = 300)"
