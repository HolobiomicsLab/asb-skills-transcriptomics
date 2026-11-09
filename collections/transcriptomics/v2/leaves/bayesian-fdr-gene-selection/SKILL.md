---
name: bayesian-fdr-gene-selection
description: Use when a fitted BUSseq SingleCellExperiment object is available and
  the goal is to rank genes by posterior cell-type-effect indicator mass and control
  Bayesian FDR at a stated alpha (e.g. 0.05) to call intrinsic genes.
when_to_use_negative:
- The input is already a corrected/imputed expression matrix rather than a fitted
  BUSseq model object — intrinsic_genes_BUSseq requires posterior indicator estimates,
  not processed counts
- Cell-type labels and batch effects are already known and validated a priori, so
  a standard bulk-style differential expression tool such as ComBat or SVA is adequate
- The number of cell types K is unknown and has not been selected via the BIC comparison
  (enumerating K, e.g. 3 to 6 as in the vignettes)
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: BUSseq
  role: Bayesian hierarchical model whose BUSseq_MCMC fit and intrinsic_genes_BUSseq
    accessor supply the spike-and-slab indicator posteriors and FDR-thresholded intrinsic
    gene set
  repo: https://github.com/songfd2018/BUSseq-Rpackage
- name: SingleCellExperiment
  role: R container that holds the BUSseq fit object (sce_BUSseqfit) passed to intrinsic_genes_BUSseq
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-020-16905-2
    title: Flexible experimental designs for valid single-cell RNA-sequencing experiments
      allowing batch effects correction
  source_content_hash: sha256:d5b445a6d508289de497f10501b82a9cb06ddb9acebc8590bbd166f4db691a95
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/songfd2018/BUSseq-Rpackage
  tools:
  - BUSseq
  - SingleCellExperiment
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# bayesian-fdr-gene-selection

## Summary

Identifies intrinsic (differentially expressed) genes that differentiate cell types from a fitted BUSseq model using posterior spike-and-slab indicator probabilities, selecting the largest gene set whose Bayesian FDR stays below a cutoff (e.g. 0.05).

## When to use

Use when a fitted BUSseq SingleCellExperiment object is available and the goal is to rank genes by posterior cell-type-effect indicator mass and control Bayesian FDR at a stated alpha (e.g. 0.05) to call intrinsic genes.

## When NOT to use

- The input is already a corrected/imputed expression matrix rather than a fitted BUSseq model object — intrinsic_genes_BUSseq requires posterior indicator estimates, not processed counts
- Cell-type labels and batch effects are already known and validated a priori, so a standard bulk-style differential expression tool such as ComBat or SVA is adequate
- The number of cell types K is unknown and has not been selected via the BIC comparison (enumerating K, e.g. 3 to 6 as in the vignettes)

## Inputs

- Fitted BUSseq model object wrapped in a SingleCellExperiment (e.g. BUSseqfits_example for the simulated and hematopoietic datasets)
- Read count matrix from multiple scRNA-seq batches (e.g. mouse hematopoietic study raw counts) with batch and cell-type metadata
- Bayesian FDR cutoff alpha (0.05 in the paper, set via the fdr argument of BUSseq_MCMC)

## Outputs

- Ranked list of intrinsic genes with posterior indicator mass kappa0 passing the Bayesian FDR threshold (e.g. 1,419 genes for the hematopoietic data)
- Gene table written to intrinsic_genes.csv
- Downstream KEGG enrichment results (51 enriched pathways on the hematopoietic run)

## How to apply

Fit the BUSseq hierarchical model with BUSseq_MCMC, which exposes the fdr setting in its signature, on the G-by-N read count matrix organized as a SingleCellExperiment. Apply intrinsic_genes_BUSseq(sce_BUSseqfit) to extract posterior estimates of the indicator probabilities xi_gk from the normal-mixture spike-and-slab prior on cell-type effects beta_gk with latent indicators L_gk. Rank genes by decreasing cumulative indicator mass kappa0 and choose the largest kappa0 whose estimated Bayesian FDR remains below alpha = 0.05. Rationale: BUSseq infers these indicators jointly with clustering and batch correction, so intrinsic-gene calls inherit the posterior uncertainty of the cell-type assignments rather than treating cluster labels as fixed. On the paper's simulation, this procedure recovered all 500 planted intrinsic genes at a true FDR of about 2% when the cutoff was set to 5%; on the mouse hematopoietic study it returned 1,419 intrinsic genes enriching 51 KEGG pathways.

## Related tools

- **BUSseq** (Bayesian hierarchical model whose BUSseq_MCMC fit and intrinsic_genes_BUSseq accessor supply the spike-and-slab indicator posteriors and FDR-thresholded intrinsic gene set) — https://github.com/songfd2018/BUSseq-Rpackage
- **SingleCellExperiment** (R container that holds the BUSseq fit object (sce_BUSseqfit) passed to intrinsic_genes_BUSseq)

## Examples

```
intrinsic_genes_BUSseq(sce_BUSseqfit)
```

## Evaluation signals

- Simulated-data check: the selected set recovers the 500 planted intrinsic genes with true FDR of approximately 2% when the Bayesian FDR cutoff is 5%
- Hematopoietic check: the selection returns 1,419 intrinsic genes (expected value from the paper)
- KEGG enrichment of the selected hematopoietic gene set yields 51 enriched pathways
- The chosen gene count corresponds to the largest kappa0 whose estimated Bayesian FDR is still below alpha = 0.05
- Sanity check on the corrected values: after correcting batch effects, cells from the same cell type across batches should cluster together, as downstream analysis treats data as if sequenced in a single batch

## Limitations

- MCMC algorithms are computationally heavy, although most steps of the BUSseq MCMC are parallelizable (see Supplementary Note 12); a completely randomized design is also hard to implement in practice for budget/time reasons
- Clustering results may be error-prone in multi-stage pipelines, but BUSseq's joint inference of cell types mitigates false positives and false negatives
- C++ sources were only tested on Linux (Ubuntu 18.04 and CentOS 7) and R 3.6.0 in the BUSseq-1.0 build instructions

## Evidence

- [methods] intrinsic_genes_BUSseq(sce_BUSseqfit): "- `intrinsic_genes_BUSseq(sce_BUSseqfit)`"
- [methods] BUSseq_MCMC signature exposes the fdr setting: "`BUSseq_MCMC(ObservedData, n.celltypes, seed, n.cores, n.iterations, n.burnin, n.unchanged, n.output, working_dir, Drop_ind, fdr, hyper_pi, hyper_gamma0, hyper_gamma1, hyper_alpha, tau1sq, hyper_p,`"
- [discussion] BUSseq identifies intrinsic genes that differentiate cell types: "BUSseq simultaneously corrects batch effects, clusters cell types, imputes missing data, and identifies intrinsic genes that differentiate cell types."
- [readme] corrected values usable for downstream analysis: "After correcting the batch effects with BUSseq, the corrected value can be used for downstream analysis as if all cells were sequenced in a single batch."
