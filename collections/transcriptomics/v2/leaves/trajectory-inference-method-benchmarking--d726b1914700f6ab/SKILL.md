---
name: trajectory-inference-method-benchmarking
description: Use when you need to quantitatively compare a new or existing trajectory-inference
  algorithm against baseline methods on synthetic single-cell data, replicating a
  published benchmark such as Supplementary Fig. 4g,h.
when_to_use_negative:
- The question is whether CellRouter's trajectories are biologically correct on real
  data (e.g. HSPC differentiation to erythroid/myeloid/lymphoid lineages) — this skill
  evaluates method performance on synthetic data only.
- The goal is to test a method designed only for linear trajectories (Waterfall, Monocle
  1) on genuinely branched multi-state transitions — these baselines are structurally
  limited and only serve as comparison points here.
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: CellRouter
  role: Method under evaluation; identifies subpopulations via community detection
    and reconstructs high-resolution multi-state trajectories between any two chosen
    cell states
  repo: https://github.com/edroaldo/cellrouter
- name: Splatter
  role: Generates the synthetic single-cell RNA-seq count matrices used as the common
    benchmark input for all methods
- name: Waterfall
  role: Baseline trajectory method limited to reconstructing linear trajectories
- name: Monocle 1
  role: Baseline trajectory method limited to reconstructing linear trajectories
- name: Monocle 2
  role: Baseline trajectory method; addresses many challenges but is not designed
    to reconstruct trajectories between any two chosen cell states
- name: Wishbone
  role: Baseline trajectory method limited to identifying only a single branch point
- name: Diffusion Pseudotime (DPT)
  role: Baseline trajectory method that requires a priori knowledge of the number
    of branches
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-018-03214-y
    title: Reconstruction of complex single-cell trajectories using CellRouter
  source_content_hash: sha256:0a1355689287e228fc5cb06eb0fd362a3b5b60decb8acb9aa362022c3074b5a4
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
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# trajectory-inference-method-benchmarking

## Summary

Benchmarks CellRouter against established single-cell trajectory detection algorithms (Monocle 1/2, DPT, Wishbone, Waterfall) on Splatter-simulated single-cell RNA-seq data, using correlated-gene counts and lag-1 autocorrelation of gene expression along pseudotime as quantitative performance metrics.

## When to use

Use when you need to quantitatively compare a new or existing trajectory-inference algorithm against baseline methods on synthetic single-cell data, replicating a published benchmark such as Supplementary Fig. 4g,h.

## When NOT to use

- The question is whether CellRouter's trajectories are biologically correct on real data (e.g. HSPC differentiation to erythroid/myeloid/lymphoid lineages) — this skill evaluates method performance on synthetic data only.
- The goal is to test a method designed only for linear trajectories (Waterfall, Monocle 1) on genuinely branched multi-state transitions — these baselines are structurally limited and only serve as comparison points here.

## Inputs

- Splatter-simulated single-cell RNA-seq count matrices (synthetic data with known trajectory structure)
- CellRouter repository (CellRouter_Class.R and Java libraries for libdir)
- Baseline algorithm outputs (Waterfall, Monocle 1, Monocle 2, Wishbone, DPT) run on the same simulated data

## Outputs

- Per-method counts of genes with expression correlated along the reconstructed trajectory
- Per-method lag-1 autocorrelation distributions of gene expression along pseudotime
- Side-by-side comparison figure replicating Supplementary Fig. 4g,h (supplementary_fig4gh_comparison.png)

## How to apply

Install the R packages CellRouter requires ('reshape','reshape2','pheatmap','tsne','igraph','ggplot2','mclust','grid','Rtsne','cccd' plus Vennerable), source the CellRouter class, and set 'libdir' to the Java libraries inside the CellRouter folder. Generate synthetic single-cell RNA-seq count matrices with Splatter using the simulation settings described in the supplement. Run CellRouter on the simulated data — identifying subpopulations by learning the structure of the data combined with community detection, then reconstructing high-resolution multi-state trajectories between any two chosen subpopulations. Run the baselines on the same simulated data, respecting each method's structural assumptions: Waterfall and Monocle 1 for linear trajectories, Wishbone for a single branch point, Diffusion Pseudotime with an a priori number of branches, and Monocle 2. For every method, count genes whose expression is correlated along the reconstructed trajectory and compute the lag-1 autocorrelation of gene expression along pseudotime; plot the per-method distributions side by side to reproduce the layout of Supplementary Fig. 4g,h.

## Related tools

- **CellRouter** (Method under evaluation; identifies subpopulations via community detection and reconstructs high-resolution multi-state trajectories between any two chosen cell states) — https://github.com/edroaldo/cellrouter
- **Splatter** (Generates the synthetic single-cell RNA-seq count matrices used as the common benchmark input for all methods)
- **Waterfall** (Baseline trajectory method limited to reconstructing linear trajectories)
- **Monocle 1** (Baseline trajectory method limited to reconstructing linear trajectories)
- **Monocle 2** (Baseline trajectory method; addresses many challenges but is not designed to reconstruct trajectories between any two chosen cell states)
- **Wishbone** (Baseline trajectory method limited to identifying only a single branch point)
- **Diffusion Pseudotime (DPT)** (Baseline trajectory method that requires a priori knowledge of the number of branches)

## Examples

```
Rscript -e "list.of.packages <- c('reshape','reshape2','pheatmap','tsne','igraph','ggplot2','mclust','grid','Rtsne','cccd'); new.packages <- list.of.packages[!(list.of.packages %in% installed.packages()[,'Package'])]; if(length(new.packages)) install.packages(new.packages)"
```

## Evaluation signals

- The reproduced figure qualitatively matches Supplementary Fig. 4g,h in layout and content
- CellRouter recovers at least as many correlated genes as the baseline methods on the simulated data
- CellRouter achieves comparable or higher lag-1 autocorrelation of gene expression along pseudotime than Waterfall, Monocle 1/2, Wishbone, and DPT
- All baseline methods were run on the exact same Splatter-simulated count matrices used for CellRouter, ensuring a fair comparison
- Each baseline is used within its documented structural scope (e.g. Wishbone only where a single branch point applies, DPT given an a priori branch number)

## Limitations

- The benchmark's validation is qualitative ('qualitatively matches Supplementary Fig. 4g,h'); no numeric acceptance thresholds are specified in the source material.
- The specific Splatter simulation parameter settings live in the supplementary material, not the main text, so exact replication depends on retrieving them.
- Baseline methods impose structural constraints (linear trajectories, single branch point, a priori branch count) that may disadvantage them independent of algorithmic quality.

## Evidence

- [results] CellRouter outperforms baselines: [evidence span withheld — anchor status: not_found]
- [intro] Baseline limitations: "Limitations include reconstructing linear trajectories (Waterfall, Monocle 1), identifying only a single branch point (Wishbone), or requiring a priori knowledge of the number of branches (Diffusion"
- [intro] Monocle 2 limitation: "Monocle 2 addresses many of these challenges but is not designed to reconstruct trajectories between any two chosen cell states"
- [methods] Required R packages: "list.of.packages <- c('reshape','reshape2','pheatmap','tsne','igraph','ggplot2','mclust','grid','Rtsne','cccd')"
- [methods] Subpopulation identification and trajectories: "Subpopulations are then identified by a combination of learning the structure of the data and community detection algorithms"
- [readme] libdir setup: "source the CellRouter class and make sure you provide the correct directory for the Java libraries inside the folder "CellRouter", as specified by the variable 'libdir'."
