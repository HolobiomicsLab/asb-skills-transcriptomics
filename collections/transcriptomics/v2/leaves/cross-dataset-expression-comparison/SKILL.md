---
name: cross-dataset-expression-comparison
description: Use when validating that simulated scRNA-seq data reproduce a protocol-specific
  expression property (e.g., gene length bias in non-UMI data) by direct comparison
  against a matched experimental dataset.
when_to_use_negative:
- Input is already observed UMI count data and the question does not concern protocol-specific
  technical bias, since UMI data show no gene length bias to compare
- The reference dataset is not matched to the simulated protocol (e.g., comparing
  UMI simulations against a non-UMI reference), which invalidates the length-bias
  comparison
- You need to simulate batch effects or trajectory/DE ground truth rather than validate
  technical-variation properties; use DivideBatches or getDEgenes/getTrajectoryGenes
  instead
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: SymSim
  role: Provides True2ObservedCounts to convert true counts into observed read/UMI
    counts with capture efficiency, amplification, fragmentation, depth, and gene
    length bias (lenslope, nbins); ships gene_len_pool and the Th17_data.RData reference
  repo: https://github.com/YosefLab/SymSim
- name: ggplot2
  role: Plots gene length versus length-normalized expression for simulated (non-UMI
    and UMI) and real Th17 data side by side
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-019-10500-w
    title: Simulating multiple faceted variability in single cell RNA sequencing
  source_content_hash: sha256:50f6561d823dd152d5a2c0571cfe40b0f16082693d3e2c4be9922dc827878576
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YosefLab/SymSim
  tools:
  - SymSim
  - ggplot2
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# Cross-dataset expression comparison

## Summary

Compares simulated SymSim scRNA-seq data with an experimental reference dataset (the Gaublomme Th17 ss2_130cells_counts non-UMI dataset) by plotting gene length versus length-normalized expression to verify that simulated technical variation reproduces gene length bias seen in real full-length protocols.

## When to use

Use when validating that simulated scRNA-seq data reproduce a protocol-specific expression property (e.g., gene length bias in non-UMI data) by direct comparison against a matched experimental dataset.

## When NOT to use

- Input is already observed UMI count data and the question does not concern protocol-specific technical bias, since UMI data show no gene length bias to compare
- The reference dataset is not matched to the simulated protocol (e.g., comparing UMI simulations against a non-UMI reference), which invalidates the length-bias comparison
- You need to simulate batch effects or trajectory/DE ground truth rather than validate technical-variation properties; use DivideBatches or getDEgenes/getTrajectoryGenes instead

## Inputs

- True counts matrix and cell metadata from SimulateTrueCounts (e.g., output of task_002)
- gene_len_pool of human transcript lengths shipped with SymSim
- Gaublomme Th17 reference dataset (ss2_130cells_counts from Th17_data.RData in SymSim extdata)

## Outputs

- Observed count matrices (non-UMI and UMI) with the same gene x cell dimensions as the input true counts
- Gene length assignments per simulated gene
- Side-by-side ggplot2 figure of gene length vs length-normalized expression for simulated and real data (Fig. 4b analog)

## How to apply

Load the true counts matrix and cell metadata from a prior SymSim SimulateTrueCounts run, and sample gene lengths from the human gene_len_pool shipped with SymSim, since each gene needs an assigned length sampled from human transcript lengths. Run True2ObservedCounts with protocol="nonUMI" (alpha_mean=0.1, alpha_sd=0.05, depth_mean=1e5, depth_sd=3e3, with length bias controlled via lenslope and nbins), and again with protocol="UMI" (alpha_mean=0.05, alpha_sd=0.02, depth_mean=5e4, depth_sd=3e3) on the same true counts as a negative control. Load the experimental reference via system.file("extdata", "Th17_data.RData", package="SymSim") to get ss2_130cells_counts (130 IL17-expressing T helper cells, non-UMI). Compute length-normalized expression per gene for both the simulated non-UMI counts and the reference counts, then plot gene length versus length-normalized expression side by side with ggplot2, reproducing the paper's Fig. 4b comparison. The rationale is that non-UMI full-length protocols exhibit length-dependent amplification that UMI protocols do not, so the comparison isolates whether the simulation of capture efficiency, amplification, fragmentation, and sequencing depth recapitulates real technical variation.

## Related tools

- **SymSim** (Provides True2ObservedCounts to convert true counts into observed read/UMI counts with capture efficiency, amplification, fragmentation, depth, and gene length bias (lenslope, nbins); ships gene_len_pool and the Th17_data.RData reference) — https://github.com/YosefLab/SymSim
- **ggplot2** (Plots gene length versus length-normalized expression for simulated (non-UMI and UMI) and real Th17 data side by side)

## Examples

```
obs <- True2ObservedCounts(true_counts, meta_cell, nCell2sample, protocol="nonUMI", alpha_mean=0.1, alpha_sd=0.05, gene_len_pool=gene_len_pool, depth_mean=1e5, depth_sd=3e3)
```

## Evaluation signals

- The observed count matrices from True2ObservedCounts have the same gene x cell dimensions as the input true counts
- The simulated non-UMI gene length vs length-normalized expression relationship visually matches the Th17 reference (ss2_130cells_counts) pattern
- No gene length dependence appears in the UMI-protocol simulated data, mirroring the absence of length bias in UMI protocols
- Length normalization removes the positive length bias in the non-UMI comparison, consistent with the paper's Fig. 4b

## Limitations

- The simulation pipeline omits several steps, including reverse transcription and library cleaning up, so not all experimental technical effects are modeled
- Details for calculating fragmentation distributions are deferred to Supplementary Note 3 and not included in the main text
- The comparison is visual (side-by-side scatter plots), not a quantitative statistical test of bias magnitude

## Evidence

- [methods] True2ObservedCounts then simulates the library preparation and sequencing procedures, and convert the true transcript counts into observed read counts or UMI counts: "True2ObservedCounts then simulates the library preparation and sequencing procedures, and convert the true transcript counts into observed read counts or UMI counts"
- [methods] Assign gene lengths: "Each genes needs to be assigned a gene length. We sample lengths from human transcript lengths."
- [methods] Th17 reference loading: "load(system.file("extdata", "Th17_data.RData", package = "SymSim"))"
- [discussion] Gaublomme non-UMI dataset: "a non-UMI-based dataset of 130 IL17-expressing T helper cells (Th17) by Gaublomme et al."
- [readme] SymSim purpose: "SymSim (Synthetic model of multiple variability factors for Simulation) is an R package for simulation of single cell RNA-Seq data."
