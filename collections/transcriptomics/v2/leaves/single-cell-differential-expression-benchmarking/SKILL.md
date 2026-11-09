---
name: single-cell-differential-expression-benchmarking
description: Use when comparing DE methods on UMI or non-UMI scRNA-seq count data
  (synthetic via Splatter templates from Islam et al. 2011 and Klein et al. 2015,
  or published GEO datasets) to measure power and FDR control.
when_to_use_negative:
- The input is already a table of DE results or adjusted q-values rather than raw
  count data or simulations to be tested
- Zero-inflation is a property specific only to a subset of genes — the article notes
  'Tweedieverse is not architected for the situation when zero-inflation is a property
  specific only to a subset of the genes'
- The design is multi-subject / individual-level DE or differential subpopulation
  analysis, which the article states is an untested extension
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: Tweedieverse
  role: 'Proposed method under evaluation: fits CPLM and ZICP Tweedie GLMs per gene
    with library-size offset and Wald testing'
  repo: https://github.com/himelmallick/Tweedieverse
- name: BenchmarkSingleCell
  role: 'Hosted benchmarking code: SingleCell_Simulator (Splatter-based simulation)
    and SingleCell_Evaluator (apply methods, compute sensitivity/FDR/FPR); run_* scripts
    for each baseline DE method'
  repo: https://github.com/himelmallick/BenchmarkSingleCell
- name: SCRAN
  role: Estimates cell-specific library sizes used as offsets in the per-gene Tweedie
    models
- name: TweedieLabs
  role: Binder-executable tutorials demonstrating Tweedieverse DE on scRNA-seq UMI
    and non-UMI counts
  repo: https://github.com/himelmallick/TweedieLabs
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/sim.9430
    title: Differential expression of single‐cell RNA‐seq data using Tweedie models
  source_content_hash: sha256:a14b63292452737480338be5363f54c959734cfb5ebe0bf79d87ab3d0a241dbf
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/himelmallick/Tweedieverse
  tools:
  - BenchmarkSingleCell
  - Bioconductor
  - R
  - SCRAN
  - TweedieLabs
  - Tweedieverse
  - cplm
  - statmod
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# single-cell-differential-expression-benchmarking

## Summary

A systematic benchmarking skill for comparing scRNA-seq differential expression methods — Tweedie-based CPLM/ZICP models from Tweedieverse versus 10+ baselines (DESeq2, edgeR, MAST, DEsingle, etc.) — on synthetic and published UMI and read-count datasets, evaluating statistical power and false discovery rate control.

## When to use

Use when comparing DE methods on UMI or non-UMI scRNA-seq count data (synthetic via Splatter templates from Islam et al. 2011 and Klein et al. 2015, or published GEO datasets) to measure power and FDR control.

## When NOT to use

- The input is already a table of DE results or adjusted q-values rather than raw count data or simulations to be tested
- Zero-inflation is a property specific only to a subset of genes — the article notes 'Tweedieverse is not architected for the situation when zero-inflation is a property specific only to a subset of the genes'
- The design is multi-subject / individual-level DE or differential subpopulation analysis, which the article states is an untested extension

## Inputs

- Synthetic scRNA-seq UMI-count datasets generated from the Klein et al. (2015) template (Splatter via SingleCell_Simulator: nSamples 20–500, nGenes 500–2000, effectSize 1–3.5, pDE 0.1, minFracZeroes 0.25)
- Synthetic scRNA-seq read-count datasets generated from the Islam et al. (2011) template
- Published GEO datasets: brain data GSE67835, 10x PBMC data (13,713 genes x 2,638 cells), mouse kidney read counts GSE107585
- Per-cell metadata / design matrix (univariate binary 'UVB') and library sizes

## Outputs

- Per-gene marginal p-values, FDR-adjusted q-values, and fold changes for every method under every scenario
- Aggregated per-method statistical power and false discovery rate comparison table (e.g., power_fdr_comparison_results.csv)
- Lists of simulated datasets from SC.sim()

## How to apply

Load synthetic count datasets generated from the Islam et al. (2011) read-count and Klein et al. (2015) UMI-count templates plus published datasets (brain GSE67835, 10x PBMC with 13,713 genes and 2,638 cells, mouse kidney GSE107585), removing genes not expressed in at least 20% of cells. Estimate cell-specific library sizes with the SCRAN R/Bioconductor package and pass them as offsets (log link) in per-gene models. Fit Tweedieverse CPLM (UMI counts) and ZICP (read counts) per gene via profile likelihood with Newton-Raphson under constraints dispersion > 0 and index parameter p in (1, 2), then run the 10+ baseline methods using the hosted BenchmarkSingleCell code (SingleCell_Simulator and SingleCell_Evaluator R scripts). Compute per-gene Wald test p-values (asymptotic chi-squared, 1 df) and FDR-adjusted q-values, calling genes DE at alpha = 0.05. Aggregate power and false discovery rate per method across plate- and droplet-based platforms into a comparison table (e.g., power_fdr_comparison_results.csv).

## Related tools

- **Tweedieverse** (Proposed method under evaluation: fits CPLM and ZICP Tweedie GLMs per gene with library-size offset and Wald testing) — https://github.com/himelmallick/Tweedieverse
- **BenchmarkSingleCell** (Hosted benchmarking code: SingleCell_Simulator (Splatter-based simulation) and SingleCell_Evaluator (apply methods, compute sensitivity/FDR/FPR); run_* scripts for each baseline DE method) — https://github.com/himelmallick/BenchmarkSingleCell
- **SCRAN** (Estimates cell-specific library sizes used as offsets in the per-gene Tweedie models)
- **TweedieLabs** (Binder-executable tutorials demonstrating Tweedieverse DE on scRNA-seq UMI and non-UMI counts) — https://github.com/himelmallick/TweedieLabs

## Examples

```
source("Library/SingleCell_Simulator.R"); sims <- SC.sim(ZeroInflate=TRUE, nSamples=100, nGenes=1000, effectSize=2, pDE=0.1, rSeed=568910)
```

## Evaluation signals

- Reproduced power/FDR comparison table confirms Tweedieverse (CPLM/ZICP) outperforms state-of-the-art DE methods across experimental platforms in statistical power and FDR control
- Both CPLM and ZICP consistently yield high F1 and AUC scores while controlling FDR, with ZICP slightly better than CPLM on zero-inflated non-UMI counts
- All reported p-values come from Wald statistics with an asymptotic chi-squared (1 df) distribution, and DE calls use FDR-adjusted q-values < 0.05
- Gene filtering invariants: simulation datasets retain only genes expressed in at least 20% of cells; mouse kidney QC retains 1138 such genes
- Benchmark output is a table of marginal p-values, adjusted q-values, and fold changes for each feature for every scenario and every method

## Limitations

- Tweedieverse is not architected for the situation when zero-inflation is a property specific only to a subset of the genes
- No borrowed-information/shrinkage estimates for improved DE gene ranking; benefits of a regularization/pooling strategy are untested
- Interoperability between platform-specific DE methods is limited (a major concern noted in the discussion); extension to other modalities (metagenomics, spatial transcriptomics) remains a conjecture
- Version numbers for SCRAN, WGCNA, Splatter, MAST, monocle, scREhurdle, DEsingle, edgeR, DESeq2, metagenomeSeq, SC2P, and SCRIP are not stated in the article section

## Evidence

- [readme] Benchmark evaluation metrics: [evidence span withheld — anchor status: not_found]
