---
name: scrna-seq-count-data-simulation-with-batch-effects-and-dropout
description: Use when validating a batch-effects correction or cell-type clustering
  method on scRNA-seq counts where ground-truth cell-type labels, known K, batch effects,
  and dropout rates must be controlled by simulation.
when_to_use_negative:
- Data are real experimental scRNA-seq counts without known ground-truth cell-type
  labels — ARI-based validation against a simulated truth is not possible
- The experimental design is completely confounded (e.g., batch 1 measures cell types
  1 and 2 while batch 2 measures cell types 3 and 4), where the paper states no method
  is applicable
- The goal is batch correction of already-normalized or bulk data (e.g., ComBat/SVA
  bulk-style inputs) rather than raw count matrices under an identifiable design
edam_operation: http://edamontology.org/operation_3566
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_0203
tools:
- name: BUSseq
  role: Bayesian hierarchical model fit to the simulated counts via BUSseq_MCMC; provides
    BIC_BUSseq for K selection, celltypes() for labels, and estimated cell-specific
    size factors for benchmarking
  repo: https://github.com/songfd2018/BUSseq
- name: BUSseq-Rpackage
  role: R package providing BUSseq_MCMC, BIC_BUSseq, celltypes(), and size-factor
    accessors used to run the simulation study
  repo: https://github.com/songfd2018/BUSseq-Rpackage
- name: BUSseq-1.0
  role: C++ implementation with simulate_data.R-style synthetic data generation demo
    and command-line BUSseq/BUSseq_inference executables for MCMC sampling
  repo: https://github.com/songfd2018/BUSseq-1.0
- name: BUSseq_gpu
  role: CUDA C alternative for the BUSseq MCMC sampler when a GPU is available
  repo: https://github.com/Anguscgm/BUSseq_gpu
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-020-16905-2
    title: Flexible experimental designs for valid single-cell RNA-sequencing experiments
      allowing batch effects correction
  source_content_hash: sha256:019252e90e22753cb01b3cf322ea88171a438aefec5b9b561cf91bfda90bd84c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/songfd2018/BUSseq
  tools:
  - BUSseq
  - BUSseq-1.0
  - BUSseq-Rpackage
  - BUSseq_gpu
  edam_operation: http://edamontology.org/operation_3566
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# scrna-seq-count-data-simulation-with-batch-effects-and-dropout

## Summary

Generate synthetic scRNA-seq read count matrices under the paper's chain-type design (e.g., 4 batches of 300/300/200/200 cells, 5 cell types, 3000 genes) with per-batch location/scale batch effects and 26.79–31.29% dropout rates, then benchmark BUSseq's ability to recover the true number of cell types via BIC, clustering (ARI), and size-factor accuracy.

## When to use

Use when validating a batch-effects correction or cell-type clustering method on scRNA-seq counts where ground-truth cell-type labels, known K, batch effects, and dropout rates must be controlled by simulation.

## When NOT to use

- Data are real experimental scRNA-seq counts without known ground-truth cell-type labels — ARI-based validation against a simulated truth is not possible
- The experimental design is completely confounded (e.g., batch 1 measures cell types 1 and 2 while batch 2 measures cell types 3 and 4), where the paper states no method is applicable
- The goal is batch correction of already-normalized or bulk data (e.g., ComBat/SVA bulk-style inputs) rather than raw count matrices under an identifiable design

## Inputs

- Simulation design parameters: number of batches (e.g., 4), cells per batch (300/300/200/200), number of cell types (5), number of genes (3000)
- Batch-effect parameters: per-batch location and scale effects mimicking real data
- Dropout-rate parameters in the 26.79–31.29% range
- Simulated G-by-N read count matrix with true batch and cell-type labels

## Outputs

- Simulated scRNA-seq read count matrix with known ground-truth cell-type and batch labels
- BIC-per-K table from BIC_BUSseq over the enumerated K values (expected minimum at the true K=5)
- Inferred cell-type assignments from celltypes() and adjusted Rand index vs. true labels (expected ARI=1.0)
- Estimated cell-specific size factors benchmarked against DESeq, TMM, library-size, and deconvolution size factors
- Summary report (e.g., busseq_simulation_report.csv)

## How to apply

Simulate a read count matrix under the chain-type experimental design with a known number of cell types (e.g., K=5) across batches of unequal sizes (e.g., 300/300/200/200 cells, 3000 genes), injecting per-batch location/scale batch effects mimicking real data and dropout rates in the 26.79–31.29% range. Fit BUSseq to the simulated counts with BUSseq_MCMC, enumerating n.celltypes (e.g., K from 3 to 6) and running 4000 MCMC iterations per K with n.cores parallelization. Select the optimal K as the minimizer of BIC_BUSseq, then extract cell type assignments with celltypes() and compute the adjusted Rand index against the true simulated labels (expected ARI=1.0 when K=5 is recovered). Also extract the estimated cell-specific size factors and compare them against DESeq, TMM, library-size, and deconvolution size factors to judge normalization accuracy. Rationale: the paper proves identifiability under the chain-type design, so correct recovery of the simulated structure demonstrates the method separates biology from technical artifacts.

## Related tools

- **BUSseq** (Bayesian hierarchical model fit to the simulated counts via BUSseq_MCMC; provides BIC_BUSseq for K selection, celltypes() for labels, and estimated cell-specific size factors for benchmarking) — https://github.com/songfd2018/BUSseq
- **BUSseq-Rpackage** (R package providing BUSseq_MCMC, BIC_BUSseq, celltypes(), and size-factor accessors used to run the simulation study) — https://github.com/songfd2018/BUSseq-Rpackage
- **BUSseq-1.0** (C++ implementation with simulate_data.R-style synthetic data generation demo and command-line BUSseq/BUSseq_inference executables for MCMC sampling) — https://github.com/songfd2018/BUSseq-1.0
- **BUSseq_gpu** (CUDA C alternative for the BUSseq MCMC sampler when a GPU is available) — https://github.com/Anguscgm/BUSseq_gpu

## Examples

```
R --vanilla --slave < simulate_data.R
```

## Evaluation signals

- BIC_BUSseq attains its minimum at the true simulated number of cell types (K=5) among enumerated K values
- Adjusted Rand index of celltypes() assignments vs. true simulated labels equals 1.0 (perfect clustering)
- BUSseq-estimated cell-specific size factors agree most closely with the simulation truth compared to DESeq, TMM, library-size, and deconvolution size factors
- Simulated dropout rates fall in the reported 26.79–31.29% range and batch cell counts match 300/300/200/200
- MCMC runs converge with the specified settings (e.g., 4000 iterations, burn-in handled by n.burnin) across all enumerated K

## Limitations

- MCMC algorithms are computationally heavy; runtimes scale with K, iterations, and dataset size (the C++ demo took ~12 min for 4 K values on 4 cores)
- The chain-type design is required for identifiability — under completely confounded designs no method is applicable
- C++ source code in BUSseq-1.0 supports Linux operating systems only (tested on Ubuntu 18.04 and CentOS 7, R 3.6.0)

## Evidence

- [results] On the simulated 4-batch, 5-cell-type dataset, BUSseq correctly identifies the presence of five cell types among the cells: [evidence span withheld — anchor status: not_found]
- [methods] To select the optimal *K*, we need to compare the BIC values for different *K*s. In the vignettes, we enumerate *K* from 3 to 6.: "To select the optimal *K*, we need to compare the BIC values for different *K*s. In the vignettes, we enumerate *K* from 3 to 6."
- [abstract] we mathematically prove that under two more flexible and realistic experimental designs-the reference panel and the chain-type designs-true biological variability can also be separated from batch: "we mathematically prove that under two more flexible and realistic experimental designs-the reference panel and the chain-type designs-true biological variability can also be separated from batch"
- [readme] we generate a synthetic dataset in *R* by running: [evidence span withheld — anchor status: external_doc]
- [discussion] most steps of the MCMC algorithm for BUSseq are parallelizable (Supplementary Note 12).: "most steps of the MCMC algorithm for BUSseq are parallelizable (Supplementary Note 12)."
