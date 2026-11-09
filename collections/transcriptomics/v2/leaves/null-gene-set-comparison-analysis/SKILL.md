---
name: null-gene-set-comparison-analysis
description: Use when validating a spatial pathway relevance method on simulated spatial
  transcriptomics data with known ground-truth gene sets, needing a null-gene-set
  separation metric.
when_to_use_negative:
- Targeted (panel-based) gene data with limited numbers of genes — GSDensity is explicitly
  not suitable for targeted gene data having limited numbers of genes
- Questions about cell-type pathway coordination in non-spatial scRNA-seq — use compute.kld
  on the MCA co-embedding instead of spatial KLD
- Datasets lacking spatial coordinates, since the activity-weighted 2D cell density
  cannot be computed
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: GSDensity
  role: Computes MCA co-embedding (compute.mca), nearest-neighbor cell-gene graph
    (compute.nn.edges, nn.use = 300), per-cell PAL scores via random walk with restart
    (run.rwr / run.rwr.list, restart = 0.75), and spatial KLD permutation tests (compute.spatial.kld
    / compute.spatial.kld.df)
  repo: https://github.com/qingnanl/gsdensity
- name: CelliD
  role: 'MCA-based benchmarking method: provides the alternative PAL-style gene set
    ranking against which GSDensity''s delta-KLD separation is compared'
  repo: https://bioconductor.org/packages/devel/bioc/html/CelliD.html
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-023-44206-x
    title: Pathway centric analysis for single-cell RNA-seq and spatial transcriptomics
      data with GSDensity
  source_content_hash: sha256:571b1429b73c0038cbd959a4174e8aa048c8d7740fddaaaca1565f6c4de687a4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/qingnanl/gsdensity
  tools:
  - CelliD
  - GSDensity
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# null-gene-set-comparison-analysis

## Summary

Benchmark whether GSDensity can separate ground-truth spatially relevant gene sets from matched random gene sets by comparing spatial KLD statistics (delta-KLD) across SRTsim-simulated ST spatial patterns (hotspot, hotspot-with-gradient, streak, gradient).

## When to use

Use when validating a spatial pathway relevance method on simulated spatial transcriptomics data with known ground-truth gene sets, needing a null-gene-set separation metric.

## When NOT to use

- Targeted (panel-based) gene data with limited numbers of genes — GSDensity is explicitly not suitable for targeted gene data having limited numbers of genes
- Questions about cell-type pathway coordination in non-spatial scRNA-seq — use compute.kld on the MCA co-embedding instead of spatial KLD
- Datasets lacking spatial coordinates, since the activity-weighted 2D cell density cannot be computed

## Inputs

- SRTsim-simulated spatial transcriptomics count matrices with spatial coordinates for four spatial organization modes (hotspot, hotspot-with-gradient, streak, gradient)
- Ground-truth spatially relevant gene sets and matched random gene sets
- Gene set collections for benchmarking (e.g., MSigDB GO Biological Process lists)

## Outputs

- Per-gene-set spatial KLD statistics and permutation p-values per simulation mode
- Delta-KLD separation values between ground-truth and random gene sets per mode
- Results table with adjusted p-values (validation criterion: adj. p < 0.05 for ground-truth sets in all modes)
- Benchmark comparison of GSDensity vs CelliD PAL-based spatial relevance

## How to apply

Simulate four ST spatial patterns (hotspot, hotspot-with-gradient, streak, gradient) with SRTsim using the settings in qingnanl/GSDensity_manuscript_code, embedding ground-truth spatially relevant gene sets plus matched random gene sets into each count matrix. Co-embed cells and genes with compute.mca (MCA) and build the cell-gene nearest-neighbor graph with compute.nn.edges using nn.use = 300. Compute per-cell PAL scores for every gene set via run.rwr / run.rwr.list with the default restart = 0.75, and in parallel with CelliD's MCA-based ranking for benchmarking. For each gene set in each mode, run compute.spatial.kld / compute.spatial.kld.df on the activity-weighted 2D cell density with n.times permutations to obtain KLD statistics and p-values. Compute delta-KLD per simulation mode as the separation between ground-truth and random gene sets; validation requires ground-truth gene sets to show significantly higher spatial KLD than random gene sets (adjusted p < 0.05) in all four modes.

## Related tools

- **GSDensity** (Computes MCA co-embedding (compute.mca), nearest-neighbor cell-gene graph (compute.nn.edges, nn.use = 300), per-cell PAL scores via random walk with restart (run.rwr / run.rwr.list, restart = 0.75), and spatial KLD permutation tests (compute.spatial.kld / compute.spatial.kld.df)) — https://github.com/qingnanl/gsdensity
- **CelliD** (MCA-based benchmarking method: provides the alternative PAL-style gene set ranking against which GSDensity's delta-KLD separation is compared) — https://bioconductor.org/packages/devel/bioc/html/CelliD.html

## Examples

```
ce <- compute.mca(object = sim_st); el <- compute.nn.edges(coembed = ce, nn.use = 300); cv <- run.rwr.list(el, gene_set_list, cells, restart = 0.75); kld <- compute.spatial.kld(activity_weighted_density, n.times = 100)
```

## Evaluation signals

- In each of the four simulation modes (hotspot, hotspot-with-gradient, streak, gradient), ground-truth spatially relevant gene sets show significantly higher spatial KLD than random gene sets with adjusted p < 0.05
- Delta-KLD computed per mode is positive and reproduces the reported benchmark separation between ground-truth and random gene sets
- GSDensity PAL scores distinguish ground-truth from random gene sets when CelliD is run in parallel as a benchmarking control
- The KLD permutation test uses the specified n.times permutations and returns both KLD statistics and p-values for every gene set in each mode
- The nearest-neighbor graph is built with nn.use = 300 and RWR uses the default restart = 0.75, matching the manuscript settings

## Limitations

- GSDensity is applicable to spatial whole transcriptomic data but is not suitable for targeted gene data having limited numbers of genes; imputation of unmeasured genes could be a strategy but is not implemented
- Neighborhood size is an important parameter when constructing the cell-gene graph, so results depend on the nn.use choice
- MCA embedding step requires large memory (~25 Gb for a 40,000-cell dataset)

## Evidence

- [intro] the spatial relevance of a pathway is reflected by the density of cells in the two-dimensional image, weighted by their pathway activity scores: "the spatial relevance of a pathway is reflected by the density of cells in the two-dimensional image, weighted by their pathway activity scores"
- [methods] Now we provide statistical test for spatial relevance of gene sets with 'compute.spatial.kld' and 'compute.spatial.kld.df' functions.: "Now we provide statistical test for spatial relevance of gene sets with 'compute.spatial.kld' and 'compute.spatial.kld.df' functions."
- [intro] Pathway activity for each cell can be calculated using network propagation in a nearest-neighbor cell-gene graph, with pathway genes used as seeds for random walks: "Pathway activity for each cell can be calculated using network propagation in a nearest-neighbor cell-gene graph, with pathway genes used as seeds for random walks"
- [readme] we use multiple correspondence analsysis (MCA) to co-embed cells and genes (thanks to the CelliD package) and then investigate the 'density' of the gene sets of interest in the MCA space: "we use multiple correspondence analsysis (MCA) to co-embed cells and genes (thanks to the CelliD package) and then investigate the 'density' of the gene sets of interest in the MCA space"
