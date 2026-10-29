---
name: bic-based-model-selection-over-cell-type-number
description: Use when fitting BUSseq to a multi-batch scRNA-seq count matrix and the
  true number of cell types K is unknown, so the latent subtype count must be chosen
  by comparing BIC values across candidate K.
when_to_use_negative:
- The number of cell types is already known or fixed by the experimental design, so
  model selection over K is unnecessary
- The data are not raw scRNA-seq read counts (e.g., already normalized or corrected
  values), since BUSseq models the count nature and dropout directly
- The experimental design is completely confounded (e.g., batch 1 measures cell types
  1 and 2 only, batch 2 measures cell types 3 and 4 only), where no method, including
  BUSseq, is applicable
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2269
tools:
- name: BUSseq
  role: Bayesian hierarchical model whose MCMC fit (BUSseq_MCMC) is run for each candidate
    K and whose BIC (BIC_BUSseq) is compared to select the optimal number of cell
    types
  repo: https://github.com/songfd2018/BUSseq-Rpackage
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-020-16905-2
    title: Flexible experimental designs for valid single-cell RNA-sequencing experiments
      allowing batch effects correction
  source_content_hash: sha256:7efee024e9e330d7216db6f1dd9a015c2742e0a6ae7add8d96148858fb482809
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/songfd2018/BUSseq-Rpackage
  tools:
  - BUSseq
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# bic-based-model-selection-over-cell-type-number

## Summary

Selects the optimal number of cell types K in BUSseq by enumerating candidate K values (e.g., K=3 to 6), running the BUSseq MCMC fit for each K, and choosing the K that minimizes the Bayesian Information Criterion (BIC).

## When to use

Use when fitting BUSseq to a multi-batch scRNA-seq count matrix and the true number of cell types K is unknown, so the latent subtype count must be chosen by comparing BIC values across candidate K.

## When NOT to use

- The number of cell types is already known or fixed by the experimental design, so model selection over K is unnecessary
- The data are not raw scRNA-seq read counts (e.g., already normalized or corrected values), since BUSseq models the count nature and dropout directly
- The experimental design is completely confounded (e.g., batch 1 measures cell types 1 and 2 only, batch 2 measures cell types 3 and 4 only), where no method, including BUSseq, is applicable

## Inputs

- Observed G-by-N scRNA-seq read count matrix (e.g., 3000 genes x 1000 cells in the 4-batch simulation)
- Batch dimension information (number of batches and per-batch cell counts, e.g., 300/300/200/200 cells)
- Dropout indicator matrix (Drop_ind) identifying zero counts attributable to dropout events
- List of candidate K values to enumerate (e.g., K = 3, 4, 5, 6)

## Outputs

- BIC value per candidate K (BIC-per-K table)
- Selected optimal K with minimum BIC (expected K=5 for the simulated chain-type dataset)
- BUSseqfits object per K from BUSseq_MCMC containing posterior cell type assignments

## How to apply

Fit BUSseq_MCMC to the observed G-by-N read count matrix separately for each candidate K (the vignettes enumerate K from 3 to 6), specifying n.iterations, n.burnin, n.cores, seed, and the dropout indicator (Drop_ind) — e.g., 4000 MCMC iterations with n.cores parallelized for each K. Compute the BIC for each fitted model with BIC_BUSseq and select the K with the minimum BIC value. Validate the selected K by extracting cell type assignments via celltypes() and comparing against known labels if available (e.g., adjusted Rand index on simulated data, where the chain-type design simulation with 4 batches and 5 cell types recovers K=5 with ARI=1.0). Rationale: since cell types are unknown in scRNA-seq, the subtype count is a latent model dimension that must be inferred by model comparison rather than fixed a priori.

## Related tools

- **BUSseq** (Bayesian hierarchical model whose MCMC fit (BUSseq_MCMC) is run for each candidate K and whose BIC (BIC_BUSseq) is compared to select the optimal number of cell types) — https://github.com/songfd2018/BUSseq-Rpackage

## Examples

```
BUSseq_MCMC(ObservedData, n.celltypes=k, seed=123, n.cores=4, n.iterations=4000, n.burnin=1000, working_dir=wd, Drop_ind=dropout_ind); BIC_BUSseq(BUSseqfits)
```

## Evaluation signals

- The BIC-versus-K plot shows a clear minimum at the selected K (expected minimum at K=5 for the simulated 5-cell-type chain-type dataset)
- Adjusted Rand index of inferred celltypes() assignments against true simulated labels equals 1.0 at the selected K
- BIC table covers the full enumeration range (K=3 to 6 in the vignettes) with a fitted BUSseq_MCMC model for every candidate K
- MCMC convergence at each K: run with adequate n.iterations (e.g., 4000), n.burnin, and n.unchanged settings, and a fixed seed for reproducibility

## Limitations

- Enumerating K requires refitting MCMC per K value; MCMC algorithms are well known for heavy computation load, though most steps are parallelizable via n.cores
- A completely confounded design (batches each measuring disjoint cell type sets) admits no applicable method, so BIC selection cannot recover true K in that setting
- The C++ source code supports Linux operating systems only (tested on Ubuntu 18.04 and CentOS 7)

## Evidence

- [methods] To select the optimal *K*, we need to compare the BIC values for different *K*s. In the vignettes, we enumerate *K* from 3 to 6.: "To select the optimal *K*, we need to compare the BIC values for different *K*s. In the vignettes, we enumerate *K* from 3 to 6."
- [readme] When we do not the number of true cell type number *K*, we run MCMC sampling for different *K*s and select the optimal *K* to obtain the minimum BIC.: [evidence span withheld — anchor status: external_doc]
- [discussion] Although MCMC algorithms are well-known for heavy computation load, fortunately, the computational complexity of BUSseq is: "Although MCMC algorithms are well-known for heavy computation load, fortunately, the computational complexity of BUSseq is"
- [intro] for completely confounded designs—for example one in which batch 1 measures cell type 1 and 2, whereas batch 2 measures cell type 3 and 4—no method is applicable: "for completely confounded designs—for example one in which batch 1 measures cell type 1 and 2, whereas batch 2 measures cell type 3 and 4—no method is applicable"
