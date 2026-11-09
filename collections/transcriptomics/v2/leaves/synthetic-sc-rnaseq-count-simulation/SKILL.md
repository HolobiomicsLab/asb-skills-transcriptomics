---
name: synthetic-sc-rnaseq-count-simulation
description: Use when benchmarking scRNA-seq differential expression methods on synthetic
  data with known ground truth (pDE genes), e.g. comparing power/FDR across UMI (CPLM)
  vs non-UMI (ZICP) platform regimes.
when_to_use_negative:
- Your goal is DE testing on a real dataset with an unknown ground truth — simulation
  is only for controlled benchmarking where truth labels (pDE genes) are required
- The target platform differs sharply from the UMI/read-count templates available
  (e.g. spatial or single-nucleus data with distinct sparsity profiles) — the Tweedieverse
  discussion notes no validation in other settings
- Zero-inflation affects only a subset of genes in your target data — the paper notes
  Tweedieverse 'is not architected for the situation when zero-inflation is a property
  specific only to a subset of the genes'
edam_operation: http://edamontology.org/operation_0307
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3307
tools:
- name: Tweedieverse
  role: Candidate DE method (CPLM for UMI counts, ZICP for read counts) evaluated
    on the simulated datasets via Wald test with FDR adjustment at alpha = 0.05
  repo: https://github.com/himelmallick/Tweedieverse
- name: BenchmarkSingleCell
  role: 'Hosts the benchmarking code: SingleCell_Simulator (Splatter-based SC.sim),
    SingleCell_Evaluator (power/FDR metrics), and run_* wrappers for the >10 baseline
    DE methods'
  repo: https://github.com/himelmallick/BenchmarkSingleCell
- name: Splatter
  role: Underlying simulator used by SingleCell_Simulator to generate synthetic scRNA-seq
    and bulk RNA-seq data
- name: SCRAN
  role: Estimates cell-specific library sizes incorporated as offsets in the per-gene
    models fitted to simulated counts
- name: SingleCell_Evaluator
  role: Computes sensitivity, FDR, and FPR metrics across simulated scenarios for
    each DE method
  repo: https://github.com/himelmallick/BenchmarkSingleCell
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/sim.9430
    title: Differential expression of single‐cell RNA‐seq data using Tweedie models
  source_content_hash: sha256:8e732f3af637f84e64172c1c614751cdc0997c4e7766bd66bf8bc31bb1df520c
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/himelmallick/Tweedieverse
  tools:
  - BenchmarkSingleCell
  - Bioconductor
  - R
  - SCRAN
  - SingleCell_Evaluator
  - Splatter
  - Tweedieverse
  - cplm
  - statmod
  edam_operation: http://edamontology.org/operation_0307
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3307
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# Synthetic scRNA-seq count simulation for DE method benchmarking

## Summary

Generates synthetic single-cell RNA-seq count datasets using Splatter-based templates (Islam et al. 2011 for read counts, Klein et al. 2015 for UMI counts) with tunable zero-inflation, effect size, and DE proportion, enabling controlled evaluation of differential expression methods in statistical power and FDR control.

## When to use

Use when benchmarking scRNA-seq differential expression methods on synthetic data with known ground truth (pDE genes), e.g. comparing power/FDR across UMI (CPLM) vs non-UMI (ZICP) platform regimes.

## When NOT to use

- Your goal is DE testing on a real dataset with an unknown ground truth — simulation is only for controlled benchmarking where truth labels (pDE genes) are required
- The target platform differs sharply from the UMI/read-count templates available (e.g. spatial or single-nucleus data with distinct sparsity profiles) — the Tweedieverse discussion notes no validation in other settings
- Zero-inflation affects only a subset of genes in your target data — the paper notes Tweedieverse 'is not architected for the situation when zero-inflation is a property specific only to a subset of the genes'

## Inputs

- Template dataset: Islam et al. (2011) read-count scRNA-seq dataset (for non-UMI simulations)
- Template dataset: Klein et al. (2015) UMI-count scRNA-seq dataset (for UMI simulations)
- Simulation parameters: nSamples, nGenes, effectSize, ZeroInflate, metadataType='UVB', minFracZeroes, pDE, nIterations, rSeed

## Outputs

- List of simulated scRNA-seq count datasets (from SC.sim / SingleCell_Simulator)
- Table of marginal p-values, FDR-adjusted q-values, and fold change for each feature for every scenario and every method (from SingleCell_Evaluator)
- Aggregated power/FDR method-by-method comparison table (e.g. power_fdr_comparison_results.csv)

## How to apply

Simulate counts with SC.sim() from the BenchmarkSingleCell SingleCell_Simulator (Splatter-based), varying nSamples (20–500 cells), nGenes (500–2000), effectSize (1–3.5), zeroInflate=True, pDE=0.1, minFracZeroes=0.25, repeated over 100 iterations with fixed seed rSeed=568910 for reproducibility. Use the Islam et al. (2011) dataset as the template for synthetic read counts and the Klein et al. (2015) dataset as the template for synthetic UMI counts, so that simulations inherit realistic platform-specific zero inflation and dispersion. After simulation, filter out genes not expressed in at least 20% of cells, estimate cell-specific library sizes with the SCRAN R/Bioconductor package, and pass them as offsets in per-gene models. Feed the simulated count matrix plus a univariate binary (UVB) metadata design into each candidate DE method (CPLM, ZICP, DESeq2, edgeR, MAST, etc.) and record marginal p-values, FDR-adjusted q-values, and fold changes per feature per scenario via SingleCell_Evaluator.

## Related tools

- **Tweedieverse** (Candidate DE method (CPLM for UMI counts, ZICP for read counts) evaluated on the simulated datasets via Wald test with FDR adjustment at alpha = 0.05) — https://github.com/himelmallick/Tweedieverse
- **BenchmarkSingleCell** (Hosts the benchmarking code: SingleCell_Simulator (Splatter-based SC.sim), SingleCell_Evaluator (power/FDR metrics), and run_* wrappers for the >10 baseline DE methods) — https://github.com/himelmallick/BenchmarkSingleCell
- **Splatter** (Underlying simulator used by SingleCell_Simulator to generate synthetic scRNA-seq and bulk RNA-seq data)
- **SCRAN** (Estimates cell-specific library sizes incorporated as offsets in the per-gene models fitted to simulated counts)
- **SingleCell_Evaluator** (Computes sensitivity, FDR, and FPR metrics across simulated scenarios for each DE method) — https://github.com/himelmallick/BenchmarkSingleCell

## Examples

```
source('Library/SingleCell_Simulator.R'); sim <- SC.sim(ZeroInflate=TRUE, metadataType='UVB', nSamples=100, nGenes=1000, effectSize=2, minFracZeroes=0.25, pDE=0.1, nIterations=100, rSeed=568910)
```

## Evaluation signals

- Simulated datasets are reproducible: rerunning SC.sim with rSeed=568910 yields identical counts across the 100 iterations
- Zero-inflation is present in simulated data (ZeroInflate=True) with minFracZeroes=0.25 minimum fraction of zeros per gene
- The known ground truth is enforced by design: pDE=0.1, so ~10% of the nGenes (500/1000/2000) are truly differentially expressed
- Genes not expressed in at least 20% of cells have been removed before model fitting
- Reproduced power/FDR table confirms Tweedieverse (CPLM/ZICP) outperforms baseline DE methods (DESeq2, edgeR, MAST, etc.) in statistical power while controlling FDR at alpha = 0.05

## Limitations

- Simulations are template-bound: only the Islam et al. (2011) and Klein et al. (2015) datasets are used as templates, limiting platform diversity
- Only univariate binary (UVB) metadata designs are simulated in the published benchmark; multi-subject/longitudinal designs (RandomEffect=False) are not exercised
- Extensions to spatial transcriptomics or other modalities remain unvalidated conjectures in the paper's discussion

## Evidence

- [readme] Simulator script role: [evidence span withheld — anchor status: not_found]
- [readme] Evaluator metrics: [evidence span withheld — anchor status: not_found]
