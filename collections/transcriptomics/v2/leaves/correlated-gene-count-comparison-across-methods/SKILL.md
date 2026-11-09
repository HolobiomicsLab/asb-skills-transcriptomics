---
name: correlated-gene-count-comparison-across-methods
description: Use when comparing trajectory-inference methods on identical single-cell
  RNA-seq input (real or Splatter-simulated) and you need quantitative smoothness
  metrics — correlated-gene counts and lag-1 autocorrelation — per method.
when_to_use_negative:
- The trajectories are not comparable because methods were run on different input
  datasets or different gene/cell preprocessing — correlated-gene counts are only
  meaningful on identical matrices
- A method requires trajectory topology the data do not support (e.g. DPT needs an
  a priori number of branches; Wishbone allows only a single branch point) — results
  would reflect the method's structural constraint rather than trajectory quality
- The goal is real-data biological interpretation (e.g. HSPC lineage discovery) rather
  than a controlled synthetic-data method benchmark
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: CellRouter
  role: Method under test; reconstructs multi-state trajectories between subpopulations
    on the simulated data
  repo: https://github.com/edroaldo/cellrouter
- name: Waterfall
  role: Baseline with linear-trajectory limitation used for correlated-gene-count
    comparison
- name: Monocle 1
  role: Baseline with linear-trajectory limitation used for correlated-gene-count
    comparison
- name: Monocle 2
  role: Baseline; cannot reconstruct trajectories between any two chosen cell states,
    included for comparison
- name: Wishbone
  role: Baseline limited to a single branch point, included for comparison
- name: Diffusion Pseudotime (DPT)
  role: Baseline requiring a priori knowledge of the number of branches, included
    for comparison
- name: Splatter
  role: Generates the synthetic single-cell RNA-seq count matrices used as the common
    benchmark input
- name: R
  role: 'Environment for running CellRouter (packages: reshape, reshape2, pheatmap,
    tsne, igraph, ggplot2, mclust, grid, Rtsne, cccd) and computing the metrics'
  repo: https://github.com/edroaldo/cellrouter
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-018-03214-y
    title: Reconstruction of complex single-cell trajectories using CellRouter
  source_content_hash: sha256:99eb3512f16b0d1d4ee8b00cdf4c55aa63ddd0b19b4d4b64f5220c7a40edf909
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/edroaldo/cellrouter
  tools:
  - CellRouter
  - Diffusion Pseudotime (DPT)
  - Monocle 1
  - Monocle 2
  - R
  - Splatter
  - Waterfall
  - Wishbone
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# Correlated-gene-count comparison across trajectory-inference methods

## Summary

Benchmark trajectory-inference algorithms (CellRouter vs Waterfall, Monocle 1, Monocle 2, Wishbone, Diffusion Pseudotime) on the same single-cell dataset by counting genes whose expression correlates along each reconstructed trajectory and computing lag-1 autocorrelation of gene expression along pseudotime, as used for the Splatter synthetic-data comparison in Supplementary Fig. 4g,h.

## When to use

Use when comparing trajectory-inference methods on identical single-cell RNA-seq input (real or Splatter-simulated) and you need quantitative smoothness metrics — correlated-gene counts and lag-1 autocorrelation — per method.

## When NOT to use

- The trajectories are not comparable because methods were run on different input datasets or different gene/cell preprocessing — correlated-gene counts are only meaningful on identical matrices
- A method requires trajectory topology the data do not support (e.g. DPT needs an a priori number of branches; Wishbone allows only a single branch point) — results would reflect the method's structural constraint rather than trajectory quality
- The goal is real-data biological interpretation (e.g. HSPC lineage discovery) rather than a controlled synthetic-data method benchmark

## Inputs

- Splatter-simulated single-cell RNA-seq count matrix (simulation settings from the CellRouter supplement)
- Reconstructed pseudotime orderings/trajectories from CellRouter, Waterfall, Monocle 1, Monocle 2, Wishbone, and DPT run on the same input

## Outputs

- Per-method counts of genes with expression correlated along the reconstructed trajectory
- Per-method lag-1 autocorrelation distributions of gene expression along pseudotime
- Side-by-side comparison figure replicating Supplementary Fig. 4g,h (e.g. supplementary_fig4gh_comparison.png)

## How to apply

First generate the common benchmark input, e.g. synthetic single-cell RNA-seq count matrices simulated with Splatter using the simulation settings described in the CellRouter supplement. Run each method on the same matrix: CellRouter (identifying subpopulations by learning the data structure with community detection, then reconstructing high-resolution multi-state trajectories between subpopulations), plus baselines with their known structural constraints — Waterfall and Monocle 1 (linear trajectories only), Wishbone (single branch point), Diffusion Pseudotime (requires the number of branches a priori), and Monocle 2. For each method's reconstructed pseudotime, count genes with expression significantly correlated along the trajectory and compute the lag-1 autocorrelation of gene expression along pseudotime, so higher values indicate smoother trajectories. Plot the correlated-gene count and lag-1 autocorrelation distributions per method side by side, replicating the layout of Supplementary Fig. 4g,h. The rationale is that the article's baselines differ in trajectory topology they can express, so a common metric (correlated genes, autocorrelation) lets smoothness be compared regardless of each method's branching assumptions.

## Related tools

- **CellRouter** (Method under test; reconstructs multi-state trajectories between subpopulations on the simulated data) — https://github.com/edroaldo/cellrouter
- **Waterfall** (Baseline with linear-trajectory limitation used for correlated-gene-count comparison)
- **Monocle 1** (Baseline with linear-trajectory limitation used for correlated-gene-count comparison)
- **Monocle 2** (Baseline; cannot reconstruct trajectories between any two chosen cell states, included for comparison)
- **Wishbone** (Baseline limited to a single branch point, included for comparison)
- **Diffusion Pseudotime (DPT)** (Baseline requiring a priori knowledge of the number of branches, included for comparison)
- **Splatter** (Generates the synthetic single-cell RNA-seq count matrices used as the common benchmark input)
- **R** (Environment for running CellRouter (packages: reshape, reshape2, pheatmap, tsne, igraph, ggplot2, mclust, grid, Rtsne, cccd) and computing the metrics) — https://github.com/edroaldo/cellrouter

## Examples

```
list.of.packages <- c('reshape','reshape2','pheatmap','tsne','igraph','ggplot2','mclust','grid','Rtsne','cccd'); install.packages(setdiff(list.of.packages, installed.packages()[,"Package"]))
```

## Evaluation signals

- The reproduced figure qualitatively matches Supplementary Fig. 4g,h in layout and distribution shapes
- CellRouter recovers at least as many correlated genes as the baseline methods on the simulated data
- CellRouter's lag-1 autocorrelation of gene expression along pseudotime is comparable to or higher than baselines, consistent with the claim that CellRouter trajectories are smoother
- All methods were run on exactly the same simulated count matrix — identical gene and cell sets per method
- Both metrics (correlated-gene counts and lag-1 autocorrelation) are reported as distributions per method, not single averages

## Limitations

- The evaluation is on Splatter synthetic data with supplement-defined simulation settings; real-data benchmarks may behave differently
- Baseline methods have structural constraints (linear trajectories, single branch point, a priori branch number) that confound the comparison of smoothness metrics
- CellRouter was tested in Unix-based systems or Mac and not in Windows, per the README
- The reproduced figure is expected to match qualitatively, not quantitatively — no numeric reproduction threshold is given

## Evidence

- [supplementary] Across these analyses, including the Splatter synthetic data comparison, trajectories identified by CellRouter are smoother and outperform the previously described algorithms by the evaluated metrics.: [evidence span withheld — anchor status: not_found]
- [intro] Limitations include reconstructing linear trajectories (Waterfall, Monocle 1), identifying only a single branch point (Wishbone), or requiring a priori knowledge of the number of branches (Diffusion: "Limitations include reconstructing linear trajectories (Waterfall, Monocle 1), identifying only a single branch point (Wishbone), or requiring a priori knowledge of the number of branches (Diffusion"
- [methods] Subpopulations are then identified by a combination of learning the structure of the data and community detection algorithms: "Subpopulations are then identified by a combination of learning the structure of the data and community detection algorithms"
- [readme] list.of.packages <- c('reshape','reshape2','pheatmap','tsne','igraph','ggplot2','mclust','grid','Rtsne','cccd'): "list.of.packages <- c('reshape','reshape2','pheatmap','tsne','igraph','ggplot2','mclust','grid','Rtsne','cccd')"
