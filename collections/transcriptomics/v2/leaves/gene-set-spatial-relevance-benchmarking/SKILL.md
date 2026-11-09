---
name: gene-set-spatial-relevance-benchmarking
description: Use when validating a pathway activity scoring method's ability to detect
  spatially relevant gene sets, by benchmarking delta-KLD separation between true
  and random gene sets across simulated ST spatial organization patterns (hotspot,
  gradient, streak).
when_to_use_negative:
- Input is targeted gene-panel ST data with limited numbers of genes — GSDensity is
  explicitly not suitable and imputation of unmeasured genes is suggested but not
  implemented
- The goal is detecting coordinated (heterogeneous) gene sets in non-spatial scRNA-seq
  without a 2D spatial map — use the non-spatial compute.kld workflow instead
- Simulation of spatial patterns is not possible or no ground-truth spatially relevant
  gene sets exist to compare against random sets
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: GSDensity
  role: 'Primary method under test: compute.mca co-embedding, compute.nn.edges graph
    (nn.use = 300), run.rwr PAL scoring, and compute.spatial.kld / compute.spatial.kld.df
    spatial relevance testing on activity-weighted 2D cell density'
  repo: https://github.com/qingnanl/gsdensity
- name: CelliD
  role: 'Benchmark comparator: MCA-based gene set ranking used in parallel with GSDensity
    PAL scores to assess spatial relevance discrimination'
  repo: https://bioconductor.org/packages/devel/bioc/html/CelliD.html
- name: SRTsim
  role: Simulates the four ST spatial organization patterns (hotspot, hotspot-with-gradient,
    streak, gradient) embedding ground-truth and random gene sets
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-023-44206-x
    title: Pathway centric analysis for single-cell RNA-seq and spatial transcriptomics
      data with GSDensity
  source_content_hash: sha256:e0a3286197b35bbda854106861bbd84de2c86f047763f8868b48ba2c276e7f17
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/qingnanl/gsdensity
  tools:
  - CelliD
  - GSDensity
  - SRTsim
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# gene-set-spatial-relevance-benchmarking

## Summary

Benchmark whether a gene set activity scoring method (GSDensity PAL vs CelliD) can distinguish ground-truth spatially relevant gene sets from matched random gene sets in spatial transcriptomics data, using delta-KLD separation on simulated ST spatial patterns.

## When to use

Use when validating a pathway activity scoring method's ability to detect spatially relevant gene sets, by benchmarking delta-KLD separation between true and random gene sets across simulated ST spatial organization patterns (hotspot, gradient, streak).

## When NOT to use

- Input is targeted gene-panel ST data with limited numbers of genes — GSDensity is explicitly not suitable and imputation of unmeasured genes is suggested but not implemented
- The goal is detecting coordinated (heterogeneous) gene sets in non-spatial scRNA-seq without a 2D spatial map — use the non-spatial compute.kld workflow instead
- Simulation of spatial patterns is not possible or no ground-truth spatially relevant gene sets exist to compare against random sets

## Inputs

- SRTsim-simulated spatial transcriptomics count matrices with spatial coordinates for four modes (hotspot, hotspot-with-gradient, streak, gradient)
- Ground-truth spatially relevant gene sets and matched random gene sets
- Simulation settings from the GSDensity manuscript code repository
- Gene set collections (e.g., MSigDB/GO) formatted as named lists

## Outputs

- Per-gene-set spatial KLD statistics and permutation p-values from compute.spatial.kld / compute.spatial.kld.df
- Delta-KLD separation values per simulation mode (ground-truth vs random gene sets)
- Benchmark results table with adjusted p-values (threshold p.adj < 0.05) for GSDensity and CelliD

## How to apply

Simulate four ST spatial organization modes (hotspot, hotspot-with-gradient, streak, gradient) with SRTsim using settings from the GSDensity manuscript repository, embedding ground-truth spatially relevant gene sets plus matched random gene sets. Load each simulated count matrix with its spatial coordinates, co-embed cells and genes with compute.mca (MCA), and build the cell-gene nearest-neighbor graph with compute.nn.edges (nn.use = 300). Compute per-cell PAL scores for every gene set via run.rwr / run.rwr.list (restart = 0.75 default) in GSDensity, and in parallel with CelliD's MCA-based ranking for benchmarking. For each gene set in each mode, run compute.spatial.kld / compute.spatial.kld.df on the activity-weighted 2D cell density with n.times permutations to obtain KLD statistics and p-values; compute delta-KLD per mode as the ground-truth vs random separation. Validation: in each of the four modes, ground-truth gene sets must show significantly higher spatial KLD than random gene sets (adjusted p < 0.05).

## Related tools

- **GSDensity** (Primary method under test: compute.mca co-embedding, compute.nn.edges graph (nn.use = 300), run.rwr PAL scoring, and compute.spatial.kld / compute.spatial.kld.df spatial relevance testing on activity-weighted 2D cell density) — https://github.com/qingnanl/gsdensity
- **CelliD** (Benchmark comparator: MCA-based gene set ranking used in parallel with GSDensity PAL scores to assess spatial relevance discrimination) — https://bioconductor.org/packages/devel/bioc/html/CelliD.html
- **SRTsim** (Simulates the four ST spatial organization patterns (hotspot, hotspot-with-gradient, streak, gradient) embedding ground-truth and random gene sets)

## Examples

```
ce <- compute.mca(object = sim_st); el <- compute.nn.edges(coembed = ce, nn.use = 300); cv <- run.rwr.list(el, gene_set_list, cells, restart = 0.75); kld <- compute.spatial.kld.df(cv, coords, n.times = 100)
```

## Evaluation signals

- In each of the four simulated modes, ground-truth spatially relevant gene sets show significantly higher spatial KLD than random gene sets (adjusted p < 0.05)
- Delta-KLD (ground-truth vs random separation) is positive and substantial in every mode for GSDensity PAL scores
- CelliD benchmark run in parallel reproduces the reported relative performance for comparison
- compute.spatial.kld permutation p-values are stable across the chosen n.times permutation count
- Simulated data reproduce the SRTsim patterns generated with the manuscript repository settings (sergio/simulation scripts in qingnanl/GSDensity_manuscript_code)

## Limitations

- GSDensity is applicable to spatial whole transcriptomic data but is not suitable for targeted gene data having limited numbers of genes; imputation of unmeasured genes could be a strategy but is not implemented
- The size of the neighborhood is an important parameter when constructing the cell-gene graph (nn.use = 300 in this workflow), so results depend on this choice
- The MCA embedding step has a large memory cost (~25 Gb for a 40,000-cell dataset)

## Evidence

- [intro] the spatial relevance of a pathway is reflected by the density of cells in the two-dimensional image, weighted by their pathway activity scores: "the spatial relevance of a pathway is reflected by the density of cells in the two-dimensional image, weighted by their pathway activity scores"
- [methods] Now we provide statistical test for spatial relevance of gene sets with 'compute.spatial.kld' and 'compute.spatial.kld.df' functions.: "Now we provide statistical test for spatial relevance of gene sets with 'compute.spatial.kld' and 'compute.spatial.kld.df' functions."
- [readme] el <- compute.nn.edges(coembed = ce, nn.use = 300): "el <- compute.nn.edges(coembed = ce, nn.use = 300)"
- [discussion] GSDensity uses network propagation to estimate the pathway relevance at single-cell level, while CelliD performs hypergeometric test: "GSDensity uses network propagation to estimate the pathway relevance at single-cell level, while CelliD performs hypergeometric test"
- [discussion] GSDensity is applicable to spatial whole transcriptomic data but is not suitable for targeted gene data having limited numbers of genes.: "GSDensity is applicable to spatial whole transcriptomic data but is not suitable for targeted gene data having limited numbers of genes."
