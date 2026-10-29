---
name: single-cell-variant-read-count-processing
description: Use when scRNA-seq somatic SNV read counts (Rs, Xs) and bulk WES counts
  (Rb, Xb) must be subsetted to mutations with estimable bursting kinetics before
  running Canopy2 tree inference.
when_to_use_negative:
- Input is raw sequencing data (FASTQ/BAM) rather than count matrices — run the somatic/germline
  variant calling pre-processing pipeline from the Zenodo repository first
- The goal is clonal-tree inference itself (get_trees/get_best_tree/BIC selection)
  rather than preparing the read-count inputs for it
- Working with methods that do not consume mutation-specific bursting-kinetics estimates,
  such as Cardelino with a guide clonal tree
tools:
- name: Canopy2
  role: R package providing get_burstiness_bpsc(), get_burstiness_scale(), the filter
    code pattern, and pre-processed sample datasets (GBM2, GBM9, GBM10, BC03, BC07)
  repo: https://github.com/annweideman/canopy2
- name: R
  role: Runtime environment for executing the canopy2 filtering and bursting-kinetics
    estimation functions
  repo: https://github.com/annweideman/canopy2
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1007/s12561-024-09466-1
    title: 'Canopy2: Tumor Phylogeny Inference by Bulk DNA and Single-Cell RNA Sequencing'
  source_content_hash: sha256:c716f4030fefaadb35de22821338c151817c3bc8a6916ff686acddf08b7af259
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/annweideman/canopy2
  tools:
  - Canopy
  - Canopy2
  - Cardelino
  - R
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# single-cell-variant-read-count-processing

## Summary

Filters and assembles the mutation-by-cell (Rs, Xs) and mutation-by-bulk (Rb, Xb) read count matrices required by Canopy2, retaining only mutations with estimable bursting kinetics and pairing them with mutation-specific gene activation/deactivation rates (alpha, beta).

## When to use

Use when scRNA-seq somatic SNV read counts (Rs, Xs) and bulk WES counts (Rb, Xb) must be subsetted to mutations with estimable bursting kinetics before running Canopy2 tree inference.

## When NOT to use

- Input is raw sequencing data (FASTQ/BAM) rather than count matrices — run the somatic/germline variant calling pre-processing pipeline from the Zenodo repository first
- The goal is clonal-tree inference itself (get_trees/get_best_tree/BIC selection) rather than preparing the read-count inputs for it
- Working with methods that do not consume mutation-specific bursting-kinetics estimates, such as Cardelino with a guide clonal tree

## Inputs

- Rs: M x N (mutation by single-cell) matrix of alternative read counts from single-cell RNA sequencing data
- Xs: M x N (mutation by single-cell) matrix of total read counts (benign + mutated) from scRNA-seq
- Rb: M x T (mutation by bulk sample) matrix of alternative read counts from bulk DNA WES
- Xb: M x T matrix of total read counts (benign + mutated) from bulk DNA WES
- Canopy2 package sample datasets, e.g. GBM10_preproc / GBM10_postproc, GBM2, GBM9, BC03, BC07

## Outputs

- Filtered Rs, Xs, Rb, Xb matrices restricted to mutations with estimable bursting kinetics
- alpha and beta: 1 x M vectors of mutation-specific gene activation and deactivation rates
- Count objects ready for get_trees() with kappa=1 and tau=999

## How to apply

Start from the post-processed Rs (M x N alternative read counts from scRNA-seq), Xs (M x N total read counts), and bulk WES Rb/Xb matrices produced by the Zenodo pre-processing pipeline (zenodo.org/record/7931384). Subset rows to mutations for which gene burstiness can be estimated, using the code pattern `mut.nonzero <- which(rowSums(Rs0)!=0)` for mutations with non-zero alternative counts across all cells and `cell.nonzero <- which(colSums(Xs0)!=0)` for cells with non-zero total counts across all mutations. Estimate the 1 x M vectors of mutation-specific gene activation (alpha) and deactivation (beta) rates with get_burstiness_bpsc() (recommended BPSC methodology) or get_burstiness_scale() (SCALE methodology). Set sequencing-error hyperparameters kappa=1 and tau=999 so kappa/(kappa+tau)=0.001, matching the reported 0.1% per-nucleotide NGS error rate. The resulting filtered matrices plus alpha/beta are the inputs to get_trees() for MCMC-based clonal tree inference on patients such as BC03, BC07, GBM9, and GBM10.

## Related tools

- **Canopy2** (R package providing get_burstiness_bpsc(), get_burstiness_scale(), the filter code pattern, and pre-processed sample datasets (GBM2, GBM9, GBM10, BC03, BC07)) — https://github.com/annweideman/canopy2
- **R** (Runtime environment for executing the canopy2 filtering and bursting-kinetics estimation functions) — https://github.com/annweideman/canopy2

## Examples

```
mut.nonzero <- which(rowSums(Rs0)!=0); cell.nonzero <- which(colSums(Xs0)!=0); param.out <- get_burstiness_bpsc(counts = GBM10_preproc$G)
```

## Evaluation signals

- All retained mutation rows have non-zero total alternative read counts across cells (rowSums(Rs)!=0) and all retained cell columns have non-zero total read counts across mutations (colSums(Xs)!=0)
- alpha and beta are 1 x M vectors whose length equals the number of mutations in the filtered Rs/Xs matrices
- kappa/(kappa+tau) equals 0.001 (0.1% per-nucleotide error) with the default kappa=1, tau=999
- Downstream get_trees() accepts the assembled inputs without dimension mismatch between Rs/Xs (M x N) and Rb/Xb (M x T)

## Limitations

- Only mutations for which gene burstiness can be estimated can be included; the rest must be subsetted out before inference
- Estimating alpha and beta from germline mutations within the same gene body as somatic mutations is optional and user-dependent
- A known BiocFileCache/dbplyr compatibility bug can cause get_gene_expression() (which internally uses biomaRt::useEnsembl()) to fail with 'Failed to collect lazy table'; the README workaround is BiocManager::install("BiocFileCache")

## Evidence

- [methods] Subset alternative and total read counts to include only mutations for which gene burstiness can be estimated.: "Subset alternative and total read counts to include only mutations for which gene burstiness can be estimated."
- [methods] identify mutations (rows) with non-zero alternative read counts across all cells: "# identify mutations (rows) with non-zero alternative read counts across all cells"
- [methods] burstiness estimation functions: "One can use function `get_burstiness_bpsc()` or `get_burstiness_scale()` to estimate `alpha` and `beta`."
- [readme] Rs matrix definition: "`Rs`:   $M \times N$ (mutation by single-cell) matrix of alternative read counts from single-cell RNA sequencing data."
- [readme] sequencing error defaults: [evidence span withheld — anchor status: external_doc]
- [methods] pre-processing pipeline location: "A sample pipeline that can be used to preprocess these data is located in our Zenodo repository at https://zenodo.org/record/7931384"
