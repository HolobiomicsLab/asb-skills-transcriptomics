---
name: model-selection-via-bic
description: Use when the true number of cell subtypes is unknown and you are fitting
  BUSseq to multi-batch scRNA-seq count data (e.g., LUAD cell lines profiled by CEL-seq2,
  10x Chromium, and Drop-seq) and must choose K objectively before final clustering.
when_to_use_negative:
- When K is known a priori (e.g., benchmarking against known cell-line identities
  where K is fixed at 3 to compute ARI against labels)
- When a completely confounded experimental design applies (each batch measures disjoint
  cell types) — no method, including BIC-based selection, is applicable
- When data are already clustered and only batch-effect correction or imputation is
  required
edam_operation: http://edamontology.org/operation_2948
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: BUSseq
  role: Fits the Bayesian hierarchical model via BUSseq_MCMC for each candidate K
    and provides BIC for model selection
  repo: https://github.com/songfd2018/BUSseq
- name: BUSseq-Rpackage
  role: R interface providing BUSseq_MCMC for enumerating K and comparing BIC values
  repo: songfd2018/BUSseq-Rpackage
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-020-16905-2
    title: Flexible experimental designs for valid single-cell RNA-sequencing experiments
      allowing batch effects correction
  source_content_hash: sha256:f5ec57b078749f8f1a2457c271060bb743403654ea04fb7f7f08a8cb1d54e63f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/songfd2018/BUSseq
  tools:
  - BUSseq
  - BUSseq-Rpackage
  - LIGER
  - MNN
  - Scanorama
  - Seurat
  edam_operation: http://edamontology.org/operation_2948
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# model-selection-via-bic

## Summary

Selects the number of cell types K for BUSseq Bayesian hierarchical-model fits by enumerating candidate K values (e.g., 3 to 6), running the BUSseq MCMC sampler for each, and comparing BIC values to pick the K with the minimum BIC.

## When to use

Use when the true number of cell subtypes is unknown and you are fitting BUSseq to multi-batch scRNA-seq count data (e.g., LUAD cell lines profiled by CEL-seq2, 10x Chromium, and Drop-seq) and must choose K objectively before final clustering.

## When NOT to use

- When K is known a priori (e.g., benchmarking against known cell-line identities where K is fixed at 3 to compute ARI against labels)
- When a completely confounded experimental design applies (each batch measures disjoint cell types) — no method, including BIC-based selection, is applicable
- When data are already clustered and only batch-effect correction or imputation is required

## Inputs

- Gene-by-cell raw read count matrix restricted to common highly variable genes (e.g., 2,267 HVGs) across multiple scRNA-seq batches (CEL-seq2, 10x Chromium, Drop-seq)
- Candidate range of K values (e.g., K = 3 to 6)
- MCMC settings: seed, n.iterations (e.g., 500), n.burnin, n.cores, working directory

## Outputs

- BIC value for each candidate K
- Scatter plot of number of cell types K versus BIC values
- Optimal K with minimum BIC (four clusters for the LUAD dataset)
- Fitted BUSseq model (BUSseqfit object) per candidate K for downstream posterior inference

## How to apply

Assemble the gene-by-cell read count matrix over the common highly variable genes (e.g., 2,267 HVGs across the three LUAD cell lines) and run BUSseq_MCMC separately for each candidate K, enumerating K from 3 to 6 in parallel on multiple cores (~2 minutes per K for 500 iterations). For each fit, compute the BIC and generate a scatter plot of K versus BIC, then select the K achieving the minimum BIC — in the LUAD benchmark this correctly identifies four clusters covering the three cell lines plus an additional cluster. Once K is chosen, refit with the selected K (or fix K=3 to match known labels for benchmarking) and proceed to posterior inference. The rationale is that BIC penalizes model complexity, so the minimum-BIC K balances fit against over-parameterization of the Bayesian model.

## Related tools

- **BUSseq** (Fits the Bayesian hierarchical model via BUSseq_MCMC for each candidate K and provides BIC for model selection) — https://github.com/songfd2018/BUSseq
- **BUSseq-Rpackage** (R interface providing BUSseq_MCMC for enumerating K and comparing BIC values) — songfd2018/BUSseq-Rpackage

## Examples

```
BUSseq_MCMC(ObservedData, n.celltypes, seed, n.cores, n.iterations, n.burnin, n.unchanged, n.output, working_dir, Drop_ind, fdr, hyper_pi, hyper_gamma0, hyper_gamma1, hyper_alpha, tau1sq, hyper_p, ...)
```

## Evaluation signals

- A BIC value is produced for every K in the enumerated range (3–6) and a K-versus-BIC scatter plot can be drawn
- The selected K is the one with the minimum BIC across all fitted candidates
- For the LUAD sc_mixology benchmark, selection should yield four clusters covering the three cell lines (HCC827, H1975, H2228)
- Refitting at the selected K and computing ARI against known labels should match published values (BUSseq K=3 ARI 0.993) within tolerance
- MCMC runs use a fixed seed so that repeated selection runs reproduce the same BIC ranking

## Limitations

- MCMC algorithms are computationally heavy; enumerating multiple K values multiplies runtime (though most MCMC steps are parallelizable across cores)
- If no candidate K in the enumerated range contains the true value, minimum-BIC selection will be misspecified
- Runtimes (~2 min per K for 500 iterations) were reported on 4 Intel i7-6700HQ cores; larger datasets need proportionally more compute

## Evidence

- [methods] BIC comparison for optimal K selection: "To select the optimal *K*, we need to compare the BIC values for different *K*s. In the vignettes, we enumerate *K* from 3 to 6."
- [readme] Minimum-BIC rule from the BUSseq demo: [evidence span withheld — anchor status: external_doc]
- [readme] Diagnostic plot of K versus BIC: [evidence span withheld — anchor status: external_doc]
- [discussion] Parallelizable MCMC supports multi-K enumeration: "most steps of the MCMC algorithm for BUSseq are parallelizable (Supplementary Note 12)."
