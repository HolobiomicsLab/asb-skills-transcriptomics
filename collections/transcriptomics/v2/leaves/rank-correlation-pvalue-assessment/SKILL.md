---
name: rank-correlation-pvalue-assessment
description: Use when evaluating a differential expression detection method against
  simulated data where true per-gene log fold changes (logFC_theoretical from getDEgenes)
  are known and ranking quality, not just binary detection, must be quantified.
when_to_use_negative:
- No gold-standard true log fold changes are available (e.g. real experimental data
  without a simulated ground truth)
- The task is clustering or trajectory inference benchmarking rather than differential
  expression detection
- Only a binary DE/non-DE verdict is needed and ranking of non-DE genes by effect
  size is irrelevant (AUROC alone suffices)
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3523
tools:
- name: SymSim
  role: Simulates true and observed single-cell RNA-Seq counts and provides getDEgenes
    to produce the gold-standard logFC_theoretical used in the correlation
  repo: https://github.com/YosefLab/SymSim
- name: RJAGS
  role: Tool catalogued for this article (Gibbs sampler for kinetic parameter estimation),
    not directly part of the p-value correlation metric
  repo: https://github.com/YosefLab/SymSim
- name: PCA
  role: Dimensionality-reduction tool catalogued for the article's clustering benchmark,
    not part of the DE p-value assessment
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-019-10500-w
    title: Simulating multiple faceted variability in single cell RNA sequencing
  source_content_hash: sha256:06987eacf3c7a9ec9895b2acfcececa5ae080de45aad854205df6965809332a3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YosefLab/SymSim
  tools:
  - PCA
  - RJAGS
  - SymSim
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3523
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# Rank-correlation p-value assessment

## Summary

Benchmarking metric that scores differential expression methods by the negative Spearman correlation between method-predicted p-values and gold-standard true log fold changes, complementing AUROC on the SymSim-simulated observed counts.

## When to use

Use when evaluating a differential expression detection method against simulated data where true per-gene log fold changes (logFC_theoretical from getDEgenes) are known and ranking quality, not just binary detection, must be quantified.

## When NOT to use

- No gold-standard true log fold changes are available (e.g. real experimental data without a simulated ground truth)
- The task is clustering or trajectory inference benchmarking rather than differential expression detection
- Only a binary DE/non-DE verdict is needed and ranking of non-DE genes by effect size is irrelevant (AUROC alone suffices)

## Inputs

- Observed count matrix from True2ObservedCounts (nonUMI protocol, simulated with SymSim)
- Gold-standard DE table from getDEgenes with columns nDiffEVF, logFC_theoretical, and adjusted Wilcoxon p-value on true counts
- Per-method predicted p-value vectors from edgeR, DESeq2, Wilcoxon rank-sum test, and t-test for population pairs (1v2, 1v4, 2v4)

## Outputs

- Negative Spearman correlation score per DE method per population comparison
- AUROC score per DE method per comparison
- Performance curves (negative Spearman and AUROC) versus cell number N and capture efficiency alpha_mean, plotted as in Fig. 6c-e as PNG figures

## How to apply

After converting SymSim true counts to observed counts with True2ObservedCounts (e.g. protocol='nonUMI', alpha_mean=0.1, depth_mean=1e5), run each DE method (edgeR, DESeq2, Wilcoxon rank-sum test, t-test) on population comparisons such as 1v2, 1v4, and 2v4 defined by the simulated tree (Phyla5). Retrieve gold-standard DE information with getDEgenes, recording nDiffEVF, logFC_theoretical, and the adjusted Wilcoxon p-value on true counts. For each method and comparison, compute the Spearman correlation between the method's predicted p-values and logFC_theoretical across all genes, then negate it so that larger values indicate better agreement between method calls and true effect size. Report this negative Spearman metric alongside AUROC, sweeping parameters such as cell number N and capture efficiency alpha_mean to draw performance curves (as in Fig. 6c-e).

## Related tools

- **SymSim** (Simulates true and observed single-cell RNA-Seq counts and provides getDEgenes to produce the gold-standard logFC_theoretical used in the correlation) — https://github.com/YosefLab/SymSim
- **RJAGS** (Tool catalogued for this article (Gibbs sampler for kinetic parameter estimation), not directly part of the p-value correlation metric) — https://github.com/YosefLab/SymSim
- **PCA** (Dimensionality-reduction tool catalogued for the article's clustering benchmark, not part of the DE p-value assessment)

## Examples

```
devtools::install_github('YosefLab/SymSim'); true_counts <- SimulateTrueCounts(evf_type='discrete', nevf=10, n_de_evf=9, vary='s', Sigma=0.4, phyla=Phyla5(), randseed=0); de <- getDEgenes(true_counts, pop1=1, pop2=2)
```

## Evaluation signals

- Negative Spearman correlation is close to 1 when method p-values perfectly anti-rank with true log fold change; verify computed values lie in [-1, 1] after negation
- Performance (negative Spearman and AUROC) should separate the three comparisons in the order expected from their DiffEVF/nDiffEVF counts
- Trend lines versus N and alpha_mean should match the qualitative pattern of Fig. 6c-e, with edgeR best, t-test second, Wilcoxon third
- Output figures exist as valid PNG files with DE gene set sizes plotted alongside performance curves

## Limitations

- The SymSim pipeline omits several steps, including reverse transcription and library cleanup, so simulated noise may under-represent some technical artifacts
- Gold-standard DE genes and logFC_theoretical are only defined because counts are simulated; the metric cannot be applied to unlabelled real data
- scVI imputation is only applicable to large datasets, limiting preprocessing choices for small non-UMI simulations

## Evidence

- [discussion] We use two metrics to evaluate the performance of a DE method: (a) AUROC (area under receiver operating characteristic curve): "We use two metrics to evaluate the performance of a DE method: (a) AUROC (area under receiver operating characteristic curve)"
- [methods] SimulateTrueCounts and True2ObservedCounts are the two major functions to generate datasets.: "**SimulateTrueCounts( )** and **True2ObservedCounts( )** are the two major functions to generate datasets."
- [methods] getDEgenes can be used to obtain DE information for genes given two populations.: "Function **getDEgenes( )** can be used to obtain differential expression information for genes, given two populations."
- [abstract] SymSim can be used to benchmark clustering, differential expression, and trajectory inference methods and to examine parameter effects on their performance: "We demonstrate how SymSim can be used for benchmarking methods for clustering, differential expression and trajectory inference, and for examining the effects of various parameters on their"
- [readme] SymSim is an R package for simulation of single cell RNA-Seq data; install via devtools from YosefLab/SymSim.: "SymSim (Synthetic model of multiple variability factors for Simulation) is an R package for simulation of single cell RNA-Seq data."
