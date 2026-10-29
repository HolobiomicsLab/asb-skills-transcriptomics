---
name: technical-noise-parameter-sweep
description: Use when benchmarking single-cell RNA-Seq analysis methods (e.g., differential
  expression detection) and you need to determine how performance degrades with fewer
  cells or lower capture efficiency.
when_to_use_negative:
- You are simulating UMI-based data — the sweep described here uses protocol='nonUMI';
  UMI counts have different capture/depth behavior.
- You need to add batch effects rather than vary capture efficiency — use DivideBatches()
  on the True2ObservedCounts output instead.
- Your question is experimental design for detecting a rare subpopulation (cell-number
  need), which the paper treats as a separate use case from technical-noise sweeps.
edam_operation: http://edamontology.org/operation_3562
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: SymSim
  role: Simulates true counts (SimulateTrueCounts) and converts them to observed counts
    under varied capture efficiency and depth (True2ObservedCounts) for the parameter
    sweep
  repo: https://github.com/YosefLab/SymSim
- name: PCA
  role: One of the dimensionality-reduction/evaluation methods applied to cells simulated
    from multiple discrete populations in the benchmarking pipeline
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-019-10500-w
    title: Simulating multiple faceted variability in single cell RNA sequencing
  source_content_hash: sha256:5dc60a04639b1e36c73769f1a42520631fcf7f431f605613965b2700be4ba728
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YosefLab/SymSim
  tools:
  - PCA
  - RJAGS
  - SymSim
  edam_operation: http://edamontology.org/operation_3562
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# technical-noise-parameter-sweep

## Summary

Sweeps technical simulation parameters (cell number N and mRNA capture efficiency alpha_mean) over a grid while converting SymSim true counts to observed counts, to quantify how capture efficiency and sequencing depth affect downstream method performance (e.g., DE detection AUROC).

## When to use

Use when benchmarking single-cell RNA-Seq analysis methods (e.g., differential expression detection) and you need to determine how performance degrades with fewer cells or lower capture efficiency.

## When NOT to use

- You are simulating UMI-based data — the sweep described here uses protocol='nonUMI'; UMI counts have different capture/depth behavior.
- You need to add batch effects rather than vary capture efficiency — use DivideBatches() on the True2ObservedCounts output instead.
- Your question is experimental design for detecting a rare subpopulation (cell-number need), which the paper treats as a separate use case from technical-noise sweeps.

## Inputs

- SymSim true transcript counts from SimulateTrueCounts (discrete EVF populations, Phyla5 tree)
- Gene length assignments sampled from gene_len_pool (human transcript lengths)
- Gold-standard DE gene sets from getDEgenes (nDiffEVF, logFC_theoretical, adjusted Wilcoxon p-value on true counts)
- Grid of technical parameters: cell number N and capture efficiency alpha_mean (with alpha_sd, depth_mean, depth_sd)

## Outputs

- Observed non-UMI count matrices per (N, alpha_mean) grid point
- Per-method AUROC and negative Spearman correlation (p-values vs true logFC) for each population comparison
- Plots of performance versus N and alpha together with DE gene set sizes (analogous to Fig. 6c-e)

## How to apply

Simulate true transcript counts with SimulateTrueCounts (evf_type='discrete', five populations via Phyla5, nevf=10, n_de_evf=9, vary='s', Sigma=0.4, randseed=0) and assign gene lengths by sampling without replacement from the gene_len_pool dataset of human transcript lengths. Extract gold-standard DE information per population pair with getDEgenes. Convert true counts to observed non-UMI read counts with True2ObservedCounts, sweeping cell number N and capture efficiency alpha_mean (baseline alpha_mean=0.1, alpha_sd=0.05, depth_mean=1e5, depth_sd=3e3) across a grid so each grid point is one observed dataset. Run the downstream methods (e.g., edgeR, DESeq2, Wilcoxon, t-test) on each observed dataset for each comparison, then score AUROC and the negative Spearman correlation between predicted p-values and true logFC_theoretical, plotting performance versus N and alpha (Fig. 6c-e pattern). The rationale is that SymSim explicitly models technical factors like mRNA capture efficiency, letting you study their effect without forcing them into a distributional model.

## Related tools

- **SymSim** (Simulates true counts (SimulateTrueCounts) and converts them to observed counts under varied capture efficiency and depth (True2ObservedCounts) for the parameter sweep) — https://github.com/YosefLab/SymSim
- **PCA** (One of the dimensionality-reduction/evaluation methods applied to cells simulated from multiple discrete populations in the benchmarking pipeline)

## Examples

```
devtools::install_github('YosefLab/SymSim'); true_counts <- SimulateTrueCounts(ncells_base=N, ngenes=ng, evf_type='discrete', nevf=10, n_de_evf=9, vary='s', Sigma=0.4, phyla=Phyla5(), randseed=0); obs <- True2ObservedCounts(true_counts, protocol='nonUMI', alpha_mean=0.1, alpha_sd=0.05, depth_mean=1e5, depth_sd=3e3)
```

## Evaluation signals

- Output figures exist as valid PNG files
- AUROC / negative Spearman trends separate the three population comparisons (1v2, 1v4, 2v4) in the order expected from their DiffEVF counts
- Qualitative pattern of performance-vs-N and performance-vs-alpha curves matches Fig. 6c-e
- DE gene set sizes tabulated per comparison are consistent with the gold-standard getDEgenes output
- Observed counts reproduce expected non-UMI properties (abundance of zeros, increased noise) without explicitly forcing them

## Limitations

- The simulation pipeline omits several steps, including reverse transcription and library cleaning up
- Gold standards are simulation-based; SymSim parameters, though process-based, still require choosing simulation settings
- scVI-based alternatives are only applicable to large datasets, limiting imputation choices for small sweeps

## Evidence

- [methods] True2ObservedCounts then simulates the library preparation and sequencing procedures, and convert the true transcript counts into observed read counts or UMI counts: "True2ObservedCounts then simulates the library preparation and sequencing procedures, and convert the true transcript counts into observed read counts or UMI counts"
- [methods] Each genes needs to be assigned a gene length. We sample lengths from human transcript lengths.: "Each genes needs to be assigned a gene length. We sample lengths from human transcript lengths."
- [intro] SymSim recapitulates properties of the data (e.g., high abundance of zeros or increased noise in non-UMI protocols): "SymSim recapitulates properties of the data (e.g., high abundance of zeros or increased noise in non-UMI protocols) without the need to explicitly force them as factors in a distributional model"
- [abstract] SymSim can be used for benchmarking methods for clustering, differential expression and trajectory inference, and for examining the effects of various parameters on their: "We demonstrate how SymSim can be used for benchmarking methods for clustering, differential expression and trajectory inference, and for examining the effects of various parameters on their"
- [readme] SymSim is an R package for simulation of single cell RNA-Seq data.: "SymSim (Synthetic model of multiple variability factors for Simulation) is an R package for simulation of single cell RNA-Seq data."
