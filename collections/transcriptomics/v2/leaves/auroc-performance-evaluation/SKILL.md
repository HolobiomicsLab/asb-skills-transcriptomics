---
name: auroc-performance-evaluation
description: Use when benchmarking differential expression (or similar binary gene-label)
  detection methods against simulated data with known ground-truth DE genes, and you
  need rank-based performance metrics (AUROC, negative Spearman of p-values vs logFC)
  to compare methods.
when_to_use_negative:
- The data have no gold-standard DE labels (e.g. purely real experimental data without
  simulated ground truth), so AUROC against known DE/non-DE classes cannot be computed
- The task is trajectory inference or clustering benchmarking rather than differential
  expression detection — SymSim supports those via getTrajectoryGenes and other metrics,
  not this DE scoring scheme
- Only a single method's calibration is needed without cross-method comparison, where
  calibration curves rather than rank-based AUROC would be more informative
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3517
tools:
- name: SymSim
  role: Generates the simulated true counts, observed counts, and gold-standard DE
    information (getDEgenes) that provide ground-truth labels for AUROC scoring
  repo: https://github.com/YosefLab/SymSim
- name: edgeR
  role: Tested differential expression detection method; achieved the best overall
    DE detection performance in the benchmark
- name: DESeq2
  role: Tested differential expression detection method scored by AUROC and negative
    Spearman correlation
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-019-10500-w
    title: Simulating multiple faceted variability in single cell RNA sequencing
  source_content_hash: sha256:41726cef0cd198e278c470c05fd954bb2633c29edd08ee0bcf4d773c68734995
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YosefLab/SymSim
  tools:
  - DESeq2
  - RJAGS
  - SymSim
  - edgeR
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3517
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# auroc-performance-evaluation

## Summary

Evaluates differential expression detection methods by scoring predicted p-values against SymSim gold-standard DE labels using AUROC and the negative Spearman correlation between predicted p-values and true theoretical log fold change, enabling ranking of methods such as edgeR, DESeq2, Wilcoxon, and t-test on simulated single-cell RNA-Seq data.

## When to use

Use when benchmarking differential expression (or similar binary gene-label) detection methods against simulated data with known ground-truth DE genes, and you need rank-based performance metrics (AUROC, negative Spearman of p-values vs logFC) to compare methods.

## When NOT to use

- The data have no gold-standard DE labels (e.g. purely real experimental data without simulated ground truth), so AUROC against known DE/non-DE classes cannot be computed
- The task is trajectory inference or clustering benchmarking rather than differential expression detection — SymSim supports those via getTrajectoryGenes and other metrics, not this DE scoring scheme
- Only a single method's calibration is needed without cross-method comparison, where calibration curves rather than rank-based AUROC would be more informative

## Inputs

- SymSim-simulated true transcript counts for multiple discrete populations (evf_type='discrete', tree such as Phyla5())
- Gold-standard DE gene labels from getDEgenes (nDiffEVF, logFC_theoretical, adjusted Wilcoxon p-value on true counts)
- Observed counts produced by True2ObservedCounts (non-UMI protocol) for each DE method's input
- Predicted per-gene p-values (and/or adjusted p-values) from each tested DE method

## Outputs

- Per-method AUROC values for each population comparison and simulation setting
- Per-method negative Spearman correlation between predicted p-values and true log fold change
- Performance-versus-parameter plots (AUROC and negative Spearman vs cell number N and capture efficiency alpha_mean), e.g. as PNG figures
- Method ranking (e.g. edgeR best, t-test second, Wilcoxon third)

## How to apply

Simulate true transcript counts with SymSim's SimulateTrueCounts (evf_type='discrete', nevf=10, n_de_evf=9, vary='s', Sigma=0.4, Phyla5 tree, randseed=0) so differential EVFs define the biological signal, then convert to observed non-UMI counts with True2ObservedCounts (alpha_mean=0.1, alpha_sd=0.05, depth_mean=1e5, depth_sd=3e3). Extract gold-standard DE information for population pairs (e.g. 1v2, 1v4, 2v4) with getDEgenes, recording nDiffEVF, logFC_theoretical, and the adjusted Wilcoxon p-value computed on true counts. Run each DE method (edgeR, DESeq2, Wilcoxon rank-sum, t-test) on the observed counts, then score it with two metrics: AUROC of the DE/non-DE classification, and the negative Spearman correlation between predicted p-values and true log fold change. Sweep simulation parameters (cell number N and capture efficiency alpha_mean) and plot performance versus these parameters to see how technical factors affect each method. The rationale, per the SymSim paper, is that simulations provide gold standards that real single-cell data lack, letting detection power be traced back to the number of differential EVFs defining each comparison.

## Related tools

- **SymSim** (Generates the simulated true counts, observed counts, and gold-standard DE information (getDEgenes) that provide ground-truth labels for AUROC scoring) — https://github.com/YosefLab/SymSim
- **edgeR** (Tested differential expression detection method; achieved the best overall DE detection performance in the benchmark)
- **DESeq2** (Tested differential expression detection method scored by AUROC and negative Spearman correlation)

## Examples

```
devtools::install_github('YosefLab/SymSim'); true_counts <- SimulateTrueCounts(ncells_total=1000, nGene_eqtl=0, evf_type='discrete', nevf=10, n_de_evf=9, vary='s', Sigma=0.4, phyla=Phyla5(), randseed=0)
```

## Evaluation signals

- AUROC separates the three population comparisons (1v2, 1v4, 2v4) in the order expected from their DiffEVF counts
- All comparison curves reproduce the qualitative pattern of the paper's Fig. 6c-e
- Output figures are valid PNG files
- edgeR consistently ranks at or near the top, with t-test second and Wilcoxon third across parameter sweeps
- Negative Spearman correlation values are negative of the raw Spearman(p-values, logFC_theoretical), i.e. higher means better

## Limitations

- Benchmark conclusions rest on SymSim's simulation assumptions; the pipeline omits several steps, including reverse transcription and library cleanup
- Conclusions may not transfer to UMI-based data since this sweep uses protocol='nonUMI' with low capture efficiency (alpha_mean=0.1)
- Simulation parameters must be matched to the real system of interest; abstract distribution-fitting simulators have parameters that do not directly correspond to the processes generating the observations

## Evidence

- [discussion] We use two metrics to evaluate the performance of a DE method: (a) AUROC (area under receiver operating characteristic curve): "We use two metrics to evaluate the performance of a DE method: (a) AUROC (area under receiver operating characteristic curve)"
- [methods] getDEgenes description: "Function getDEgenes( ) can be used to obtain differential expression information for genes, given two populations."
- [intro] simulation difficulty without gold standards: "it is difficult to compare them to each other and assess their performance without gold standards. One approach to address"
- [methods] SimulateTrueCounts and True2ObservedCounts: "**SimulateTrueCounts( )** and **True2ObservedCounts( )** are the two major functions to generate datasets."
- [abstract] SymSim benchmarking scope: "We demonstrate how SymSim can be used for benchmarking methods for clustering, differential expression and trajectory inference"
- [readme] SymSim README reference: "SymSim (Synthetic model of multiple variability factors for Simulation) is an R package for simulation of single cell RNA-Seq data."
