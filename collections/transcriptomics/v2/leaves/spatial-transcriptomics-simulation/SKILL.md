---
name: spatial-transcriptomics-simulation
description: 'Use when validating a spatial-pathway or spatial-relevance method on
  ST data where no ground truth exists: simulate tissue patterns with SRTsim (or SERGIO-style
  scripts from the manuscript repo), plant known spatially relevant gene sets, and
  test whether the method separates them from random sets.'
when_to_use_negative:
- Data are already real ST tissue sections with known anatomical annotations — evaluate
  spatial relevance directly rather than simulating
- The data are targeted-gene ST panels with limited gene numbers — GSDensity/simulation
  is not suitable and gene imputation would be needed first
- The benchmark target is scRNA-seq cell-pathway detection rather than spatial organization
  — use the SERGIO steady/dynamic simulation scripts instead of SRTsim spatial modes
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: SRTsim
  role: Simulates ST count matrices under four controlled spatial organization patterns
    with embedded ground-truth and random gene sets
  repo: qingnanl/GSDensity_manuscript_code
- name: GSDensity
  role: 'Method under benchmark: compute.mca, compute.nn.edges (nn.use=300), run.rwr.list,
    compute.spatial.kld/df produce the delta-KLD separation being validated'
  repo: https://github.com/qingnanl/gsdensity
- name: CelliD
  role: 'Benchmark comparator: MCA-based pathway ranking scored on the same simulated
    ST modes'
  repo: https://bioconductor.org/packages/devel/bioc/html/CelliD.html
- name: SERGIO
  role: Simulates scRNA-seq data (steady and dynamic modes) via the manuscript-repo
    scripts for the non-spatial side of the benchmark
  repo: qingnanl/GSDensity_manuscript_code
provenance:
  source_task_ids:
  - task_006
  - task_008
  - task_007
  source_papers:
  - doi: 10.1038/s41467-023-44206-x
    title: Pathway centric analysis for single-cell RNA-seq and spatial transcriptomics
      data with GSDensity
  source_content_hash: sha256:58540adf124b7c73cee92b8c7eeccbb9c50365403d493c8f41d9ce1edfe9a8e9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/qingnanl/GSDensity_manuscript_code
  tools:
  - CelliD
  - GSDensity
  - SERGIO
  - SRTsim
  - Seurat
  - SeuratData
  - future.apply
  - igraph
  - msigdbr
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 16
---

# spatial-transcriptomics-simulation

## Summary

Benchmark spatial-relevance detection by simulating spatial transcriptomics (ST) data with controlled spatial organization patterns (e.g., four SRTsim modes: hotspot, hotspot-with-gradient, streak, gradient) and embedding ground-truth spatially relevant gene sets alongside matched random gene sets, so that a method like GSDensity's delta-KLD separation can be validated against known ground truth.

## When to use

Use when validating a spatial-pathway or spatial-relevance method on ST data where no ground truth exists: simulate tissue patterns with SRTsim (or SERGIO-style scripts from the manuscript repo), plant known spatially relevant gene sets, and test whether the method separates them from random sets.

## When NOT to use

- Data are already real ST tissue sections with known anatomical annotations — evaluate spatial relevance directly rather than simulating
- The data are targeted-gene ST panels with limited gene numbers — GSDensity/simulation is not suitable and gene imputation would be needed first
- The benchmark target is scRNA-seq cell-pathway detection rather than spatial organization — use the SERGIO steady/dynamic simulation scripts instead of SRTsim spatial modes

## Inputs

- Simulation settings/deposit from qingnanl/GSDensity_manuscript_code for SRTsim spatial patterns (hotspot, hotspot-with-gradient, streak, gradient)
- Ground-truth spatially relevant gene sets and matched random gene sets to embed
- Simulated cell-by-gene count matrix with per-spot 2D spatial coordinates
- Gene set collections (e.g., GO BP, KEGG, BIOCARTA via msigdbr) for coordinated-pathway testing

## Outputs

- Per-gene-set, per-mode spatial KLD statistics and permutation-based p-values from compute.spatial.kld.df (n.times permutations)
- Delta-KLD per simulation mode quantifying separation between ground-truth and random gene sets
- Results table of PAL scores (GSDensity vs CelliD benchmark) per gene set per spatial mode
- Pass/fail validation verdict per mode (ground-truth sets significantly higher KLD, adjusted p < 0.05)

## How to apply

Simulate the target ST spatial patterns with SRTsim using the simulation settings deposited in the GSDensity manuscript code repository, embedding ground-truth spatially relevant gene sets plus matched random gene sets into each simulated count matrix with spatial coordinates. Process each simulated dataset through the method under test exactly as for real data: co-embed cells and genes with gsdensity::compute.mca, build the cell-gene nearest-neighbor graph with compute.nn.edges (nn.use = 300), and score per-cell pathway activity with run.rwr / run.rwr.list (restart = 0.75 default), benchmarking against CelliD's MCA-based ranking in parallel. Test spatial relevance per gene set per mode with compute.spatial.kld / compute.spatial.kld.df on the activity-weighted 2D cell density using n.times permutation replicates, then compute delta-KLD per simulation mode as the separation between ground-truth and random gene sets. For scRNA-seq simulation benchmarks, the same repository provides SERGIO scripts (sergio_steady_gen.R, run_sergio_code.py, sergio_dyn_gen.R) and benchmark.sergio()/benchmark.real() wrappers with designs such as '5000.gene.3ct', 1500 cells per type, and dropout settings like 'dp1'.

## Related tools

- **SRTsim** (Simulates ST count matrices under four controlled spatial organization patterns with embedded ground-truth and random gene sets) — qingnanl/GSDensity_manuscript_code
- **GSDensity** (Method under benchmark: compute.mca, compute.nn.edges (nn.use=300), run.rwr.list, compute.spatial.kld/df produce the delta-KLD separation being validated) — https://github.com/qingnanl/gsdensity
- **CelliD** (Benchmark comparator: MCA-based pathway ranking scored on the same simulated ST modes) — https://bioconductor.org/packages/devel/bioc/html/CelliD.html
- **SERGIO** (Simulates scRNA-seq data (steady and dynamic modes) via the manuscript-repo scripts for the non-spatial side of the benchmark) — qingnanl/GSDensity_manuscript_code

## Examples

```
data.sum <- benchmark.sergio(design = "5000.gene.3ct", version.n = 1, drop.out = "dp1", n.cell.types = 3, n.cells.per.type = 1500, n.genes = 5000)
```

## Evaluation signals

- In each of the four simulated modes (hotspot, hotspot-with-gradient, streak, gradient), ground-truth spatially relevant gene sets show significantly higher spatial KLD than random gene sets (adjusted p < 0.05)
- Delta-KLD computed per mode is positive and separates ground-truth from random gene-set distributions
- Permutation nulls use the specified n.times replicates; p-values are adjusted for multiple testing before calling significance
- Simulated count matrices carry both expression values and 2D spatial coordinates so compute.spatial.kld.df can build activity-weighted vs background 2D densities
- For SERGIO-side benchmarks, benchmark.sergio runs complete for the chosen design (e.g., '5000.gene.3ct', n.genes = 5000, 1500 cells/type, drop.out = 'dp1') without missing-mode errors

## Limitations

- GSDensity is applicable to spatial whole transcriptomic data but is not suitable for targeted gene data having limited numbers of genes; imputation of unmeasured genes is suggested but not implemented
- Neighborhood size (nn.use) is an important parameter when constructing the cell-gene graph, so benchmark results can be sensitive to this choice
- Memory cost is high: the MCA embedding step needs about 25 Gb for a 40,000-cell dataset
- Over- or under-correction of batch effects may introduce bias, requiring careful evaluation before pathway analysis on simulated-plus-real combinations

## Evidence

- [abstract] GSDensity can identify spatially relevant pathways in mouse brains and human tumors including those following high-order organizational patterns in the ST data: "GSDensity can identify spatially relevant pathways in mouse brains and human tumors including those following high-order organizational patterns in the ST data"
- [methods] Now we provide statistical test for spatial relevance of gene sets with 'compute.spatial.kld' and 'compute.spatial.kld.df' functions.: "Now we provide statistical test for spatial relevance of gene sets with 'compute.spatial.kld' and 'compute.spatial.kld.df' functions."
- [readme] sergio_analysis_functions.R; method.benchmarking.functions.R: a set of functions for simulated data benchmarking: [evidence span withheld — anchor status: external_doc]
- [discussion] GSDensity is applicable to spatial whole transcriptomic data but is not suitable for targeted gene data having limited numbers of genes.: "GSDensity is applicable to spatial whole transcriptomic data but is not suitable for targeted gene data having limited numbers of genes."
