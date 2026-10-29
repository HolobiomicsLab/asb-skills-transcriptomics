---
name: size-factor-normalization-benchmarking
description: Use when a simulated or real scRNA-seq read count matrix with known cell-specific
  size factors (or ground-truth cell types) is available and you need to evaluate
  whether BUSseq's cell-specific size factors outperform DESeq, TMM, library-size,
  or deconvolution normalization.
when_to_use_negative:
- When all subtypes must be present in each batch is required by your downstream method
  instead (e.g. MetaSparseKmeans), or the design is completely confounded (batches
  measure disjoint cell types) — BUSseq requires identifiable designs such as the
  chain-type or reference panel designs
- When data are already normalized expression values rather than raw counts — BUSseq
  models raw read counts and does not require a preliminary normalization
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: BUSseq
  role: Bayesian hierarchical model (BUSseq_MCMC) fitted to the read counts; its inferred
    cell-specific size factors are the primary normalization method under benchmark
  repo: https://github.com/songfd2018/BUSseq
- name: BUSseq-Rpackage
  role: R package providing BUSseq_MCMC, BIC_BUSseq, and celltypes() accessors used
    to fit the model, select K by BIC, and extract assignments and size factors
  repo: https://github.com/songfd2018/BUSseq-Rpackage
- name: DESeq
  role: Competing size-factor estimation scheme used as a normalization benchmark
    against BUSseq's size factors
- name: TMM
  role: Competing normalization scheme benchmarked against BUSseq size factors
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-020-16905-2
    title: Flexible experimental designs for valid single-cell RNA-sequencing experiments
      allowing batch effects correction
  source_content_hash: sha256:f29203a3b48514b87e488fcf8987a7d153e5f9c5027c0574455cc5a182b25024
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/songfd2018/BUSseq
  tools:
  - BUSseq
  - BUSseq-Rpackage
  - DESeq
  - TMM
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# Size-factor normalization benchmarking

## Summary

Extract the cell-specific size factors estimated by BUSseq's Bayesian hierarchical model from a simulated multi-batch scRNA-seq fit and benchmark their normalization accuracy against standard normalization schemes (DESeq, TMM, library-size, deconvolution), demonstrating that joint estimation of size factors with batch-effect correction and cell-type clustering recovers sequencing depth accurately even under dropout.

## When to use

Use when a simulated or real scRNA-seq read count matrix with known cell-specific size factors (or ground-truth cell types) is available and you need to evaluate whether BUSseq's cell-specific size factors outperform DESeq, TMM, library-size, or deconvolution normalization.

## When NOT to use

- When all subtypes must be present in each batch is required by your downstream method instead (e.g. MetaSparseKmeans), or the design is completely confounded (batches measure disjoint cell types) — BUSseq requires identifiable designs such as the chain-type or reference panel designs
- When data are already normalized expression values rather than raw counts — BUSseq models raw read counts and does not require a preliminary normalization

## Inputs

- Simulated or raw scRNA-seq read count matrix (genes x cells) with batch and cell-type metadata (e.g. count_data_[study]_v[version].txt plus dim_ and metadata files)
- True simulated cell type labels and true size factors for benchmarking
- BUSseqfit object returned by BUSseq_MCMC

## Outputs

- BUSseq-estimated cell-specific size factors per cell
- Comparison of BUSseq size factors against DESeq, TMM, library-size, and deconvolution size factors
- BIC-per-K table (e.g. K=3 to 6) with selected K
- Adjusted Rand index of inferred vs. true cell type labels (expected 1.0 in the chain-type simulation)
- busseq_simulation_report.csv summarizing BIC, ARI, and size-factor benchmarking results

## How to apply

Fit BUSseq to the raw read count matrix with BUSseq_MCMC across candidate cell-type counts (e.g. K=3 to 6 as in the vignettes, 4000 MCMC iterations, parallelized via n.cores), and select K by minimum BIC via BIC_BUSseq. Verify clustering quality by extracting cell type assignments with celltypes() and computing the adjusted Rand index against the true labels (the chain-type simulation with 4 batches of 300/300/200/200 cells, 5 cell types, 3000 genes, and 26.79–31.29% dropout yields BIC minimum at K=5 and ARI=1.0). Then extract the estimated cell-specific size factors from the BUSseqfit object and compare them against DESeq, TMM, library-size, and deconvolution size factors, writing the comparison into busseq_simulation_report.csv. Rationale: BUSseq models the count nature, overdispersion, dropout events, and cell-specific sequencing depth jointly, so its size factors are inferred without preliminary normalization, unlike the competing schemes.

## Related tools

- **BUSseq** (Bayesian hierarchical model (BUSseq_MCMC) fitted to the read counts; its inferred cell-specific size factors are the primary normalization method under benchmark) — https://github.com/songfd2018/BUSseq
- **BUSseq-Rpackage** (R package providing BUSseq_MCMC, BIC_BUSseq, and celltypes() accessors used to fit the model, select K by BIC, and extract assignments and size factors) — https://github.com/songfd2018/BUSseq-Rpackage
- **DESeq** (Competing size-factor estimation scheme used as a normalization benchmark against BUSseq's size factors)
- **TMM** (Competing normalization scheme benchmarked against BUSseq size factors)

## Examples

```
BUSseqfit <- BUSseq_MCMC(ObservedData, n.celltypes=5, n.iterations=4000, n.cores=4); BIC_BUSseq(BUSseqfit); celltypes(BUSseqfit)
```

## Evaluation signals

- BIC per candidate K is computed and its minimum falls at the true K=5 on the chain-type simulation
- Adjusted Rand index between celltypes() output and true simulated labels equals 1.0
- BUSseq size factors show lower error against the true simulated size factors than DESeq, TMM, library-size, and deconvolution size factors
- Fitted model dimension matches the simulated design (4 batches of 300/300/200/200 cells, 3000 genes)
- Results table busseq_simulation_report.csv is produced with BIC, ARI, and size-factor comparison entries populated

## Limitations

- MCMC inference is computationally heavy; the paper notes 'MCMC algorithms are well-known for heavy computation load', though most steps are parallelizable (Supplementary Note 12) and a GPU/CUDA variant exists
- The C++ sources are Linux-only (tested on Ubuntu 18.04 and CentOS 7) and code was tested on R 3.6.0
- Completely confounded designs admit no applicable method; identifiability requires reference panel or chain-type designs

## Evidence

- [discussion] BUSseq simultaneously corrects batch effects, clusters cell types, imputes missing data, and identifies intrinsic genes that differentiate cell types.: "BUSseq simultaneously corrects batch effects, clusters cell types, imputes missing data, and identifies intrinsic genes that differentiate cell types."
- [methods] Busseq R function signature: "`BUSseq_MCMC(ObservedData, n.celltypes, seed, n.cores, n.iterations, n.burnin, n.unchanged, n.output, working_dir, Drop_ind, fdr, hyper_pi, hyper_gamma0, hyper_gamma1, hyper_alpha, tau1sq, hyper_p,"
- [methods] BIC-based K selection: "To select the optimal *K*, we need to compare the BIC values for different *K*s. In the vignettes, we enumerate *K* from 3 to 6."
- [readme] README on what BUSseq accounts for: "takes care of the count data nature, the overdispersion, the dropout events, and the cell-specific sequencing depth of scRNA-seq data"
- [abstract] Evaluation claim: "We demonstrate that BUSseq outperforms existing methods with simulated and real data."
