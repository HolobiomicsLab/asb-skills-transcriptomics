---
name: single-cell-intrinsic-gene-detection
description: Use when scRNA-seq count data across multiple batches has been fitted
  with BUSseq and you need the gene set that differentiates the discovered cell types
  at a controlled Bayesian FDR cutoff.
when_to_use_negative:
- The data has not been fitted with BUSseq (no posterior spike-and-slab indicator
  estimates exist) — differential expression methods for pre-normalized or bulk data
  like ComBat/SVA are a different setting
- The experimental design is completely confounded (e.g. batch 1 measures cell types
  1 and 2, batch 2 measures cell types 3 and 4), where no method, including BUSseq,
  is applicable
- The goal is batch-effect-corrected expression matrices rather than gene selection
  — use corrected_read_counts() or imputed_read_counts() instead
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: BUSseq
  role: Bayesian hierarchical model whose spike-and-slab prior on cell-type effects
    beta_gk supplies the posterior indicator probabilities used to call intrinsic
    genes at Bayesian FDR 0.05
  repo: https://github.com/songfd2018/BUSseq
- name: SingleCellExperiment
  role: Container holding the fitted BUSseq object (sce_BUSseqfit) passed to intrinsic_genes_BUSseq()
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-020-16905-2
    title: Flexible experimental designs for valid single-cell RNA-sequencing experiments
      allowing batch effects correction
  source_content_hash: sha256:75ab1d3a35ebd1faa3d39ec5f05f937ff01bea4164a295f9e5e904c08b9f9e36
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/songfd2018/BUSseq
  tools:
  - BUSseq
  - SingleCellExperiment
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# single-cell-intrinsic-gene-detection

## Summary

Identifies intrinsic (differentially expressed) genes that differentiate cell types from a fitted BUSseq Bayesian hierarchical model, using the posterior indicator probabilities of the spike-and-slab prior on cell-type effects and controlling the Bayesian FDR (alpha = 0.05).

## When to use

Use when scRNA-seq count data across multiple batches has been fitted with BUSseq and you need the gene set that differentiates the discovered cell types at a controlled Bayesian FDR cutoff.

## When NOT to use

- The data has not been fitted with BUSseq (no posterior spike-and-slab indicator estimates exist) — differential expression methods for pre-normalized or bulk data like ComBat/SVA are a different setting
- The experimental design is completely confounded (e.g. batch 1 measures cell types 1 and 2, batch 2 measures cell types 3 and 4), where no method, including BUSseq, is applicable
- The goal is batch-effect-corrected expression matrices rather than gene selection — use corrected_read_counts() or imputed_read_counts() instead

## Inputs

- Fitted BUSseq model object (e.g. BUSseqfits_example) or SingleCellExperiment container holding the BUSseq fit
- Raw G-by-N scRNA-seq read count matrix with batch and cell-type (clustering) metadata from the BUSseq_MCMC run
- Bayesian FDR threshold alpha (e.g. 0.05, the fdr argument of BUSseq_MCMC)

## Outputs

- Ranked list of intrinsic genes selected at Bayesian FDR <= 0.05 (e.g. intrinsic_genes.csv)
- Posterior indicator probabilities xi_gk / cumulative indicator mass kappa0 per gene
- Downstream KEGG pathway enrichment results for the selected gene set

## How to apply

Fit the BUSseq model with BUSseq_MCMC (setting the fdr argument, default alpha = 0.05) on the G-by-N read count matrix, then load the fitted object (e.g. BUSseqfits_example) as a SingleCellExperiment. Apply intrinsic_genes_BUSseq() to the fit to obtain posterior estimates of the indicator probabilities xi_gk arising from the normal-mixture spike-and-slab prior on cell-type effects beta_gk with latent indicators L_gk. Rank genes by decreasing cumulative indicator mass kappa0 and select the largest gene set whose estimated Bayesian FDR remains below alpha = 0.05. Validate by comparing against known positives: in the simulation, this recovered all 500 planted intrinsic genes at ~2% true FDR, and on the mouse hematopoietic data it returned 1,419 intrinsic genes with 51 enriched KEGG pathways. Write the final gene table (e.g. intrinsic_genes.csv) for downstream enrichment or marker analysis.

## Related tools

- **BUSseq** (Bayesian hierarchical model whose spike-and-slab prior on cell-type effects beta_gk supplies the posterior indicator probabilities used to call intrinsic genes at Bayesian FDR 0.05) — https://github.com/songfd2018/BUSseq
- **SingleCellExperiment** (Container holding the fitted BUSseq object (sce_BUSseqfit) passed to intrinsic_genes_BUSseq())

## Examples

```
R: intrinsic_genes <- intrinsic_genes_BUSseq(sce_BUSseqfit); write.csv(intrinsic_genes, "intrinsic_genes.csv")
```

## Evaluation signals

- On simulated data with planted positives, the selected set recovers all 500 true intrinsic genes with a true FDR of approximately 2% at the 5% Bayesian FDR cutoff
- On the mouse hematopoietic dataset, the procedure returns exactly 1,419 intrinsic genes
- KEGG enrichment of the hematopoietic intrinsic gene set yields 51 enriched pathways, confirming biological meaningfulness
- The chosen gene set size corresponds to the largest kappa0 whose estimated Bayesian FDR stays below alpha = 0.05
- Output table (intrinsic_genes.csv) is written with genes ranked by decreasing cumulative indicator mass

## Limitations

- BUSseq relies on MCMC inference, which has a heavy computational load, although most MCMC steps are parallelizable (a GPU/CUDA implementation, BUSseq_gpu, exists)
- Cell-type clustering errors in earlier stages can propagate, leading to false positives and false negatives in intrinsic gene calls because multi-stage uncertainty is not fully modeled
- Requires an experimental design (completely randomized, reference panel, or chain-type) in which biological variability is identifiable; completely confounded designs admit no applicable method

## Evidence

- [methods] intrinsic_genes_BUSseq(sce_BUSseqfit): "- `intrinsic_genes_BUSseq(sce_BUSseqfit)`"
- [methods] BUSseq_MCMC signature exposes the fdr setting: "`BUSseq_MCMC(ObservedData, n.celltypes, seed, n.cores, n.iterations, n.burnin, n.unchanged, n.output, working_dir, Drop_ind, fdr, hyper_pi, hyper_gamma0, hyper_gamma1, hyper_alpha, tau1sq, hyper_p,`"
- [discussion] BUSseq identifies intrinsic genes that differentiate cell types: "BUSseq simultaneously corrects batch effects, clusters cell types, imputes missing data, and identifies intrinsic genes that differentiate cell types."
- [results] Simulation achieves ~2% true FDR at 5% Bayesian FDR: [evidence span withheld — anchor status: not_found]
- [readme] BUSseq fits an interpretable Bayesian hierarchical model: "our proposed `BUSseq` method fits an interpretable Bayesian hierarchical model---the Batch Effects Correction with Unknown Subtypes for scRNA seq Data (BUSseq)---to correct batch effects in the"
