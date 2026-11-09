---
name: single-cell-rna-seq-count-simulation
description: Use when you need gold-standard scRNA-seq count matrices with known population
  structure, DE genes, or parameter ground truth to benchmark clustering/DE methods
  or to plan experiments for rare-population detection.
when_to_use_negative:
- You need multi-omics, spatial, or gene-regulatory-network-driven simulation — the
  authors generalized SymSim into scMultiSim for those cases
- The input is already observed scRNA-seq data you want to cluster or DE-test directly
  rather than simulate a gold standard for
- You need to model reverse transcription and library cleanup steps, which the SymSim
  pipeline explicitly omits
edam_operation: http://edamontology.org/operation_0284
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: SymSim
  role: 'Core simulator: SimulateTrueCounts generates true transcript counts; True2ObservedCounts
    simulates library preparation and sequencing; BestMatchParams, DivideBatches,
    getDEgenes, PlotTsne, Phyla5 support the workflow'
  repo: https://github.com/YosefLab/SymSim
- name: RJAGS
  role: Gibbs-sampler MCMC (2000 iterations) to fit kinetic parameters (kon, koff,
    s) from imputed experimental data
- name: PCA
  role: Dimensionality reduction feeding PlotTsne visualization and k-means clustering
    of simulated populations
- name: scVI
  role: Alternative embedding/dimensionality reduction for clustering benchmarking
    on simulated counts
- name: SIMLR
  role: Dimensionality reduction method compared against PCA and scVI when clustering
    simulated discrete populations
- name: ggplot2
  role: Plotting simulated-vs-real comparisons, e.g., gene length vs length-normalized
    expression (Fig. 4b) and Q-Q plots
provenance:
  source_task_ids:
  - task_003
  - task_004
  - task_008
  - task_002
  - task_005
  - task_006
  source_papers:
  - doi: 10.1038/s41467-019-10500-w
    title: Simulating multiple faceted variability in single cell RNA sequencing
  source_content_hash: sha256:1cbba4baad01fa11d1c6cd7a5e138650dcf56b759f03897c3b5596b216b89e92
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YosefLab/SymSim
  tools:
  - BestMatchParams
  - PCA
  - RJAGS
  - Rtsne
  - SIMLR
  - SimulateTrueCounts
  - SymSim
  - True2ObservedCounts
  - ape
  - ggplot2
  - scVI
  edam_operation: http://edamontology.org/operation_0284
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6111111111111112
n_claims: 18
---

# single-cell-rna-seq-count-simulation

## Summary

Generate in silico single-cell RNA-seq datasets with SymSim by simulating true transcript counts from kinetic/EVF models (SimulateTrueCounts) and converting them to observed UMI or non-UMI counts (True2ObservedCounts), explicitly modeling capture efficiency, amplification/length bias, fragmentation, and sequencing depth.

## When to use

Use when you need gold-standard scRNA-seq count matrices with known population structure, DE genes, or parameter ground truth to benchmark clustering/DE methods or to plan experiments for rare-population detection.

## When NOT to use

- You need multi-omics, spatial, or gene-regulatory-network-driven simulation — the authors generalized SymSim into scMultiSim for those cases
- The input is already observed scRNA-seq data you want to cluster or DE-test directly rather than simulate a gold standard for
- You need to model reverse transcription and library cleanup steps, which the SymSim pipeline explicitly omits

## Inputs

- Phylogenetic tree of populations (e.g., Newick_ABCDE.txt via ape::read.tree or Phyla5())
- Gene lengths sampled from gene_len_pool (human transcript lengths)
- Real count matrix for parameter matching (e.g., cortex_counts.txt UMI cortex, Th17_data.RData ss2_130cells_counts)
- Simulation settings: ncells_total, ngenes, nevf, n_de_evf, evf_type, Sigma, min_popsize, i_minpop, randseed
- Technical parameters: protocol, alpha_mean/alpha_sd, depth_mean/depth_sd, lenslope, nbins

## Outputs

- True transcript count matrix (genes x cells) with cell metadata containing EVFs
- Observed count matrix (genes x cells) for UMI or non-UMI protocol with same dimensions as true counts
- Gold-standard DE gene tables (nDiffEVF, logFC_theoretical, adjusted Wilcoxon p-value) from getDEgenes
- t-SNE plots (PlotTsne on log2(true_counts+1), n_pc=20) colored by population or tree edge
- Batch-split observed count matrices from DivideBatches

## How to apply

Install SymSim via devtools::install_github('YosefLab/SymSim') after the required Bioconductor packages (IRanges, Biobase, S4Vectors, SummarizedExperiment). Call SimulateTrueCounts with a tree (e.g., Phyla5()), ngenes, nevf/n_de_evf, evf_type='discrete' or 'continuous', Sigma for within-population variability, and min_popsize/i_minpop to embed a rare leaf; for a single population, BestMatchParams can recover parameters matching a real dataset's statistics over a pre-computed simulation grid. Assign gene lengths by sampling from gene_len_pool (human transcript lengths), then run True2ObservedCounts with protocol='UMI' (e.g., alpha_mean=0.05, depth_mean=5e4) or 'nonUMI' (e.g., alpha_mean=0.1, alpha_sd=0.05, depth_mean=1e5, depth_sd=3e3, with lenslope/nbins for length bias). Optionally use DivideBatches to split observed counts into batches with added batch effects, getDEgenes to obtain gold-standard DE information for population pairs, and PlotTsne on log2(true_counts+1) with n_pc=20 to verify the simulated structure. Validate that simulated non-UMI data reproduce gene length bias seen in real data (e.g., the Gaublomme Th17 ss2_130cells_counts reference) and that UMI data do not.

## Related tools

- **SymSim** (Core simulator: SimulateTrueCounts generates true transcript counts; True2ObservedCounts simulates library preparation and sequencing; BestMatchParams, DivideBatches, getDEgenes, PlotTsne, Phyla5 support the workflow) — https://github.com/YosefLab/SymSim
- **RJAGS** (Gibbs-sampler MCMC (2000 iterations) to fit kinetic parameters (kon, koff, s) from imputed experimental data)
- **PCA** (Dimensionality reduction feeding PlotTsne visualization and k-means clustering of simulated populations)
- **scVI** (Alternative embedding/dimensionality reduction for clustering benchmarking on simulated counts)
- **SIMLR** (Dimensionality reduction method compared against PCA and scVI when clustering simulated discrete populations)
- **ggplot2** (Plotting simulated-vs-real comparisons, e.g., gene length vs length-normalized expression (Fig. 4b) and Q-Q plots)

## Examples

```
true_counts_res <- SimulateTrueCounts(ncells_total=300, min_popsize=50, i_minpop=2, ngenes=500, nevf=10, evf_type="discrete", n_de_evf=9, vary="s", Sigma=0.4, phyla=Phyla5(), randseed=0); observed <- True2ObservedCounts(true_counts=true_counts_res[[1]], protocol="nonUMI", alpha_mean=0.1, alpha_sd=0.05, depth_mean=1e5, depth_sd=3e3, gene_len=gene_len_sampled)
```

## Evaluation signals

- Observed count matrices must have the same gene x cell dimensions as the input true counts
- Simulated non-UMI gene length vs length-normalized expression should visually match the Th17 (ss2_130cells_counts) reference pattern, with no length bias in UMI simulations
- Q-Q plots from BestMatchParams should show mean expression, percent non-zero, and gene SD falling close to the identity line for the top-ranked simulation
- t-SNE of discrete simulations should show five cell groups mirroring the five leaves of the input Phyla5 tree; continuous simulations should show a connected trajectory structure
- AUROC/negative-Spearman DE benchmarks should separate population comparisons (1v2, 1v4, 2v4) in the order expected from their DiffEVF counts

## Limitations

- The simulation pipeline omits reverse transcription and library cleanup steps
- Kinetic parameter fitting assumes all cells in a cluster share the same kinetic parameters, so cells must be divided by clustering first
- Supplementary Tables 2-5 with fitted parameters and Supplementary Note 3 fragmentation details are not in the main text
- scVI is only applicable to large datasets; non-UMI data were imputed with MAGIC instead
- Learned generative models (e.g., scVI) replacing EVF/kinetics sacrifice biological interpretability of parameters

## Evidence

- [methods] SimulateTrueCounts and True2ObservedCounts: "**SimulateTrueCounts( )** and **True2ObservedCounts( )** are the two major functions to generate datasets."
- [methods] gene length assignment: "Each genes needs to be assigned a gene length. We sample lengths from human transcript lengths."
- [methods] True2ObservedCounts role: "True2ObservedCounts then simulates the library preparation and sequencing procedures, and convert the true transcript counts into observed read counts or UMI counts"
- [methods] batch effects: "DivideBatches( ) takes the output of True2ObservedCounts( ) as input and split the data into multiple batches while adding batch effect for each batch."
- [methods] installation: "devtools::install_github('YosefLab/SymSim')"
- [readme] package purpose: "SymSim (Synthetic model of multiple variability factors for Simulation) is an R package for simulation of single cell RNA-Seq data."
- [abstract] three sources of variation: "The components of the SymSim pipeline pertain to the three primary sources of variation in single cell RNA-Seq data: noise intrinsic to the process of transcription, extrinsic variation indicative of"
- [intro] recapitulates real-data properties: "SymSim recapitulates properties of the data (e.g., high abundance of zeros or increased noise in non-UMI protocols) without the need to explicitly force them as factors in a distributional model"
