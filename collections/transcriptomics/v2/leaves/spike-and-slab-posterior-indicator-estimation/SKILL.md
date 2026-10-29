---
name: spike-and-slab-posterior-indicator-estimation
description: Use when a fitted BUSseq model is available and the goal is to detect
  intrinsic genes that differentiate cell types at a controlled Bayesian false discovery
  rate (e.g., alpha = 0.05) from posterior indicator probabilities.
when_to_use_negative:
- Input is raw count matrices that have not yet been fitted by BUSseq_MCMC (no posterior
  indicator probabilities exist yet); first run the MCMC and BIC-based K selection.
- The goal is batch-effect correction, imputation, or clustering rather than differential-expression
  gene detection; use corrected_read_counts() or imputed_read_counts() instead.
- The experimental design is completely confounded (each batch measures disjoint cell
  types), where the article states no method is applicable.
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: BUSseq
  role: Provides the Bayesian hierarchical model, MCMC inference, and the intrinsic_genes_BUSseq()
    function that estimates spike-and-slab indicator probabilities and selects intrinsic
    genes at the Bayesian FDR cutoff
  repo: https://github.com/songfd2018/BUSseq-Rpackage
- name: SingleCellExperiment
  role: Container class (sce_BUSseqfit) that wraps the fitted BUSseq object passed
    to intrinsic_genes_BUSseq()
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-020-16905-2
    title: Flexible experimental designs for valid single-cell RNA-sequencing experiments
      allowing batch effects correction
  source_content_hash: sha256:7c84177f926282fd471ce7056e58f8ea6fb581ca96ce0f392fdff9b262efff4d
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
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# spike-and-slab-posterior-indicator-estimation

## Summary

Reconstructs BUSseq's Bayesian-FDR procedure for identifying intrinsic (differentially expressed) genes, using posterior estimates of spike-and-slab indicator probabilities on cell-type effects and ranking genes by cumulative indicator mass under an FDR cutoff of 0.05.

## When to use

Use when a fitted BUSseq model is available and the goal is to detect intrinsic genes that differentiate cell types at a controlled Bayesian false discovery rate (e.g., alpha = 0.05) from posterior indicator probabilities.

## When NOT to use

- Input is raw count matrices that have not yet been fitted by BUSseq_MCMC (no posterior indicator probabilities exist yet); first run the MCMC and BIC-based K selection.
- The goal is batch-effect correction, imputation, or clustering rather than differential-expression gene detection; use corrected_read_counts() or imputed_read_counts() instead.
- The experimental design is completely confounded (each batch measures disjoint cell types), where the article states no method is applicable.

## Inputs

- Fitted BUSseq model object (e.g., BUSseqfits_example) wrapped in a SingleCellExperiment container
- Bayesian FDR cutoff alpha (default 0.05, matching the fdr argument in BUSseq_MCMC)

## Outputs

- Ranked list of intrinsic genes with posterior indicator mass (kappa0) and Bayesian FDR estimates
- Selected intrinsic gene set (e.g., 1,419 genes on the hematopoietic fit)
- intrinsic_genes.csv table of selected genes

## How to apply

Load the fitted BUSseq model object (e.g., BUSseqfits_example for the simulated or hematopoietic scRNA-seq fits) into an R session with the BUSseq package and its SingleCellExperiment container. Apply intrinsic_genes_BUSseq() to the fit to obtain posterior estimates of the indicator probabilities xi_gk derived from the normal-mixture spike-and-slab prior on cell-type effects beta_gk with latent indicators L_gk. Rank genes by decreasing cumulative indicator mass kappa0 and select the largest kappa0 whose estimated Bayesian FDR stays below alpha = 0.05, the fdr setting exposed in the BUSseq_MCMC signature. On simulated data, compare the selected gene set against the 500 planted intrinsic genes to estimate the true FDR (target ~2%); on real data (hematopoietic fit), confirm the selection returns 1,419 intrinsic genes and validate biological meaning via KEGG enrichment expecting 51 enriched pathways. Write the final gene table to intrinsic_genes.csv.

## Related tools

- **BUSseq** (Provides the Bayesian hierarchical model, MCMC inference, and the intrinsic_genes_BUSseq() function that estimates spike-and-slab indicator probabilities and selects intrinsic genes at the Bayesian FDR cutoff) — https://github.com/songfd2018/BUSseq-Rpackage
- **SingleCellExperiment** (Container class (sce_BUSseqfit) that wraps the fitted BUSseq object passed to intrinsic_genes_BUSseq())

## Examples

```
intrinsic_genes <- intrinsic_genes_BUSseq(sce_BUSseqfit); write.csv(intrinsic_genes, "intrinsic_genes.csv")
```

## Evaluation signals

- On the simulated dataset, the selected set recovers all 500 planted intrinsic genes with an estimated true FDR of approximately 2% under the 5% Bayesian FDR cutoff.
- On the hematopoietic fit, the procedure returns exactly 1,419 intrinsic genes.
- KEGG enrichment on the hematopoietic intrinsic gene set yields 51 enriched pathways.
- The chosen gene set is the largest kappa0 whose estimated Bayesian FDR remains below alpha = 0.05.
- The final gene table is written to intrinsic_genes.csv with posterior indicator probabilities xi_gk available from intrinsic_genes_BUSseq(sce_BUSseqfit).

## Limitations

- MCMC algorithms are well-known for heavy computation load; fortunately, most steps of the BUSseq MCMC are parallelizable (Supplementary Note 12), but runtime can still be substantial.
- Multi-stage pipelines that ignore uncertainties in earlier clustering stages can produce false positives and false negatives in downstream gene detection.
- Requires an experimental design where biological variability is identifiable (completely randomized, reference panel, or chain-type); completely confounded designs admit no applicable method.

## Evidence

- [results] In the simulation study, controlling the Bayesian False Discovery Rate at 5% allowed BUSseq to identify all intrinsic genes that differentiate cell types, achieving a true FDR of 2%.: [evidence span withheld — anchor status: not_found]
- [methods] intrinsic_genes_BUSseq is exposed as a filter step on the fitted SingleCellExperiment object: "- `intrinsic_genes_BUSseq(sce_BUSseqfit)`"
- [discussion] BUSseq simultaneously corrects batch effects, clusters cell types, imputes missing data, and identifies intrinsic genes: "BUSseq simultaneously corrects batch effects, clusters cell types, imputes missing data, and identifies intrinsic genes that differentiate cell types."
- [methods] BUSseq_MCMC exposes the fdr parameter governing the Bayesian FDR cutoff used for intrinsic gene selection: "`BUSseq_MCMC(ObservedData, n.celltypes, seed, n.cores, n.iterations, n.burnin, n.unchanged, n.output, working_dir, Drop_ind, fdr, hyper_pi, hyper_gamma0, hyper_gamma1, hyper_alpha, tau1sq, hyper_p,"
- [readme] The BUSseq method is described in the package README as an interpretable Bayesian hierarchical model for scRNA-seq data: "our proposed `BUSseq` method fits an interpretable Bayesian hierarchical model---the Batch Effects Correction with Unknown Subtypes for scRNA seq Data (BUSseq)---to correct batch effects in the"
