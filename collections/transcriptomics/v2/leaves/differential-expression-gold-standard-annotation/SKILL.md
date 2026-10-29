---
name: differential-expression-gold-standard-annotation
description: Use when benchmarking DE detection methods on simulated scRNA-Seq data
  where ground-truth differential genes must be extracted from known true counts (e.g.
  SymSim population pairs 1v2, 1v4, 2v4) before any observed-count analysis.
when_to_use_negative:
- Input data are real experimental scRNA-Seq counts with no known ground-truth DE
  status — the skill requires simulated data with true transcript counts and known
  EVFs.
- The goal is trajectory- or clustering-method evaluation rather than DE detection
  — use getTrajectoryGenes or the clustering benchmark instead.
- Only observed counts are available without the paired true counts; the adjusted
  Wilcoxon p-value gold standard must be computed on true counts, not observed counts.
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: SymSim
  role: Simulates true and observed counts (SimulateTrueCounts, True2ObservedCounts)
    and provides getDEgenes to extract gold-standard differential expression information
    from true counts
  repo: https://github.com/YosefLab/SymSim
- name: edgeR
  role: One of the benchmarked DE detection methods scored against the gold standard;
    found to have the best overall DE detection performance
- name: DESeq2
  role: One of the four benchmarked DE detection methods evaluated against the simulated
    gold standard
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-019-10500-w
    title: Simulating multiple faceted variability in single cell RNA sequencing
  source_content_hash: sha256:35b09d1afc2a99259757b2f663787bf70f39225eadbfbe5e228a38708761a863
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YosefLab/SymSim
  tools:
  - DESeq2
  - RJAGS
  - SymSim
  - edgeR
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# differential-expression-gold-standard-annotation

## Summary

Annotates gold-standard differential expression signal from SymSim-simulated data by running getDEgenes on simulated true transcript counts (nDiffEVF, logFC_theoretical, adjusted Wilcoxon p-value), enabling AUROC and negative Spearman-correlation benchmarking of DE detection methods (edgeR, DESeq2, Wilcoxon, t-test) on the paired observed counts.

## When to use

Use when benchmarking DE detection methods on simulated scRNA-Seq data where ground-truth differential genes must be extracted from known true counts (e.g. SymSim population pairs 1v2, 1v4, 2v4) before any observed-count analysis.

## When NOT to use

- Input data are real experimental scRNA-Seq counts with no known ground-truth DE status — the skill requires simulated data with true transcript counts and known EVFs.
- The goal is trajectory- or clustering-method evaluation rather than DE detection — use getTrajectoryGenes or the clustering benchmark instead.
- Only observed counts are available without the paired true counts; the adjusted Wilcoxon p-value gold standard must be computed on true counts, not observed counts.

## Inputs

- SymSim true transcript counts from SimulateTrueCounts (evf_type="discrete", five discrete populations, Phyla5 tree, randseed=0)
- Population-pair definitions for comparison (e.g. 1v2, 1v4, 2v4)
- gene_len_pool dataset of human transcript lengths (for assigning gene lengths)

## Outputs

- Per-comparison gold-standard DE gene lists from getDEgenes with nDiffEVF, logFC_theoretical, and adjusted Wilcoxon p-value on true counts
- Tabulated DE gene set sizes for each population pair
- AUROC and negative Spearman correlation (predicted p-values vs true logFC) scores for each benchmarked DE method

## How to apply

Simulate true transcript counts for cells from five discrete populations with SimulateTrueCounts(evf_type="discrete", nevf=10, n_de_evf=9, vary="s", Sigma=0.4, phyla=Phyla5(), randseed=0) so that differentially expressed extrinsic variation factors (DiffEVF) define the biological signal. Assign gene lengths by sampling without replacement from the gene_len_pool dataset of human transcript lengths. Extract gold-standard DE information for the population pairs 1v2, 1v4, and 2v4 with getDEgenes, recording nDiffEVF, logFC_theoretical, and the adjusted Wilcoxon p-value computed on true counts, and tabulate the DE gene set sizes per pair. Only after the gold standard is fixed, convert true counts to observed counts with True2ObservedCounts (protocol="nonUMI", alpha_mean=0.1, alpha_sd=0.05, depth_mean=1e5, depth_sd=3e3) and run the candidate DE methods. Rationale: differential EVFs on true counts constitute the only noise-free reference, so the gold standard must be derived before technical simulation to avoid circularity.

## Related tools

- **SymSim** (Simulates true and observed counts (SimulateTrueCounts, True2ObservedCounts) and provides getDEgenes to extract gold-standard differential expression information from true counts) — https://github.com/YosefLab/SymSim
- **edgeR** (One of the benchmarked DE detection methods scored against the gold standard; found to have the best overall DE detection performance)
- **DESeq2** (One of the four benchmarked DE detection methods evaluated against the simulated gold standard)

## Examples

```
true_counts_res <- SimulateTrueCounts(evf_type="discrete", nevf=10, n_de_evf=9, vary="s", Sigma=0.4, phyla=Phyla5(), randseed=0); de_1v2 <- getDEgenes(true_counts_res, pop1=1, pop2=2)
```

## Evaluation signals

- Output figures exist as valid PNG files plotting AUROC and negative Spearman correlation versus cell number N and capture efficiency alpha_mean, together with the DE gene set sizes (matching the qualitative pattern of Fig. 6c-e)
- AUROC / negative Spearman trends separate the three population comparisons in the order expected from their DiffEVF counts (e.g. 1v2, 1v4, 2v4 with different DiffEVF abundance)
- Gold-standard DE gene set sizes are non-empty and tabulated consistently for all three population pairs (1v2, 1v4, 2v4)
- Adjusted Wilcoxon p-values in the gold standard are computed on true transcript counts, not observed counts
- Method ranking is reproducible across the parameter grid (edgeR best, t-test second, Wilcoxon third, consistent with prior evaluation of 36 DE methods)

## Limitations

- The simulation pipeline omits several steps, including reverse transcription and library cleaning up, so observed counts do not capture all technical artifacts of real protocols.
- The DE detection benchmark uses SymSim-simulated data; gold-standard validity depends on the simulation parameters (evf_type="discrete", n_de_evf=9) and may not transfer to other EVF regimes.
- scVI-based imputation alternatives are only applicable to large datasets, constraining some downstream preprocessing choices for small simulated datasets.

## Evidence

- [methods] getDEgenes gold standard: "Function getDEgenes( ) can be used to obtain differential expression information for genes, given two populations."
- [discussion] evaluation metrics: "We use two metrics to evaluate the performance of a DE method: (a) AUROC (area under receiver operating characteristic curve)"
- [methods] core simulation functions: "SimulateTrueCounts( ) and True2ObservedCounts( ) are the two major functions to generate datasets."
- [readme] SymSim purpose: "SymSim (Synthetic model of multiple variability factors for Simulation) is an R package for simulation of single cell RNA-Seq data."
