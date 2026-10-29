---
name: bayesian-hierarchical-model-mcmc-inference
description: Use when raw multi-batch scRNA-seq read count matrices contain unknown
  cell subtypes with batch effects and dropouts, and you need simultaneous batch correction
  and clustering without prior normalization or subtype labels.
when_to_use_negative:
- The experimental design is completely confounded (e.g., batch 1 measures only cell
  types 1-2 and batch 2 only cell types 3-4) — no method, including BUSseq, is applicable
  because biological and batch variation cannot be separated
- Subtype information is already known a priori and only supervised label transfer
  is needed (bulk-style methods like ComBat/SVA apply, or semi-supervised alternatives
  such as CellAssign)
- The input is already normalized or batch-corrected expression data rather than raw
  read counts, since BUSseq models the count nature, overdispersion, and dropouts
  of raw scRNA-seq counts directly
edam_operation: http://edamontology.org/operation_3656
edam_topics:
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_2229
tools:
- name: BUSseq
  role: Primary R package implementing the Bayesian hierarchical model, BUSseq_MCMC
    sampler, BIC_BUSseq selection, and downstream accessors (celltypes, corrected/imputed
    counts, intrinsic genes)
  repo: https://github.com/songfd2018/BUSseq-Rpackage
- name: BUSseq-1.0
  role: C++ parallel MCMC implementation invoked as a compiled binary with flags -K,
    -i, -b, -s, -c for iterations, burn-in, seed, and cores
  repo: https://github.com/songfd2018/BUSseq-1.0
- name: BUSseq_gpu
  role: CUDA C GPU-accelerated version of the same MCMC sampler, compiled with nvcc
    and run with -B, -N, -G arguments
  repo: https://github.com/Anguscgm/BUSseq_gpu
provenance:
  source_task_ids:
  - task_001
  - task_006
  - task_004
  - task_002
  source_papers:
  - doi: 10.1038/s41467-020-16905-2
    title: Flexible experimental designs for valid single-cell RNA-sequencing experiments
      allowing batch effects correction
  source_content_hash: sha256:22203fec5462780cf17c8d3136ea4785fd9b2365578eed5fa7297f8c20ab5b94
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/songfd2018/BUSseq-Rpackage
  tools:
  - BUSseq
  - BUSseq-1.0
  - BUSseq_gpu
  - ComBat
  - LIGER
  - MNN
  - RUV
  - SVA
  - Scanorama
  - Seurat
  - scMerge
  edam_operation: http://edamontology.org/operation_3656
  edam_topics:
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7894736842105263
n_claims: 19
---

# bayesian-hierarchical-model-mcmc-inference

## Summary

Fit the BUSseq interpretable Bayesian hierarchical model to multi-batch scRNA-seq read counts via MCMC (BUSseq_MCMC), simultaneously correcting batch effects, clustering unknown cell types, modeling dropouts and overdispersion, and selecting the number of cell types K by comparing BIC values.

## When to use

Use when raw multi-batch scRNA-seq read count matrices contain unknown cell subtypes with batch effects and dropouts, and you need simultaneous batch correction and clustering without prior normalization or subtype labels.

## When NOT to use

- The experimental design is completely confounded (e.g., batch 1 measures only cell types 1-2 and batch 2 only cell types 3-4) — no method, including BUSseq, is applicable because biological and batch variation cannot be separated
- Subtype information is already known a priori and only supervised label transfer is needed (bulk-style methods like ComBat/SVA apply, or semi-supervised alternatives such as CellAssign)
- The input is already normalized or batch-corrected expression data rather than raw read counts, since BUSseq models the count nature, overdispersion, and dropouts of raw scRNA-seq counts directly

## Inputs

- Genes-by-cells raw read count matrix across multiple batches (e.g., count_data_[study]_v[version].txt)
- Dimension file with total cells, gene count, number of batches, and per-batch cell counts (dim_[study]_v[version].txt)
- Batch and (optionally) cell-type metadata per cell (metadata_[study]_v[version].txt)
- Candidate number of cell types K (e.g., enumerated 3 to 6)
- MCMC settings: seed, n.cores, n.iterations, n.burnin, n.output, Drop_ind, hyperparameters (hyper_pi, hyper_gamma0, hyper_gamma1, hyper_alpha, tau1sq, hyper_p, fdr)

## Outputs

- BUSseqfits model object with posterior samples of cell-type means, overdispersion, dropout coefficients, and cell-specific size factors
- Inferred cell-type assignments per cell (celltypes())
- BIC value per candidate K for model selection (BIC_BUSseq)
- Corrected and imputed read count matrices (corrected_read_counts(), imputed_read_counts())
- Intrinsic gene list (intrinsic_genes_BUSseq()) and posterior predictive zero-rate comparison per batch

## How to apply

Assemble a genes-by-cells read count matrix with per-batch dimension and metadata files, then fit BUSseq with the BUSseq_MCMC function (BUSseq R package, installed via install_github('songfd2018/BUSseq-Rpackage')), specifying n.celltypes, seed, n.cores, n.iterations (e.g., 4000 with burn-in, or the package default ~500 iterations per K), n.burnin, and Drop_ind to enable the zero-inflated dropout component. Because the true number of subtypes is unknown, enumerate K (the vignettes enumerate K from 3 to 6), run MCMC in parallel for each K, and select the optimal K as the one with the minimum BIC from BIC_BUSseq. For alternative performance needs, the C++ parallel version (BUSseq-1.0, compiled with RngStreams and run as `BUSseq -d./ -r... -p project -v version -K K -i iterations -o output_freq -s seed -c cores`) or the CUDA GPU version (Anguscgm/BUSseq_gpu, compiled with nvcc) can be used, since most steps of the MCMC algorithm are parallelizable. After convergence, extract inferred cell-type assignments (celltypes()), corrected/imputed read counts, size factors, and intrinsic genes; validate by checking that BIC is minimized at the expected K, that the adjusted Rand index against known labels is competitive (e.g., 0.582 hematopoietic, 0.993 LUAD with K fixed at 3), and that posterior predictive zero rates match observed zero rates per batch (e.g., 0.887 vs 0.892 on MARS-seq).

## Related tools

- **BUSseq** (Primary R package implementing the Bayesian hierarchical model, BUSseq_MCMC sampler, BIC_BUSseq selection, and downstream accessors (celltypes, corrected/imputed counts, intrinsic genes)) — https://github.com/songfd2018/BUSseq-Rpackage
- **BUSseq-1.0** (C++ parallel MCMC implementation invoked as a compiled binary with flags -K, -i, -b, -s, -c for iterations, burn-in, seed, and cores) — https://github.com/songfd2018/BUSseq-1.0
- **BUSseq_gpu** (CUDA C GPU-accelerated version of the same MCMC sampler, compiled with nvcc and run with -B, -N, -G arguments) — https://github.com/Anguscgm/BUSseq_gpu

## Examples

```
BUSseq_MCMC(ObservedData, n.celltypes=5, seed=123, n.cores=4, n.iterations=4000, n.burnin=1000, Drop_ind=TRUE)
```

## Evaluation signals

- BIC is minimized at the expected K (e.g., BIC-selected K=5 on the simulated 4-batch, 5-cell-type chain-type dataset)
- Adjusted Rand index of inferred cell types vs. known labels reproduces published values (ARI=1.0 on simulation; 0.582 hematopoietic; 0.993 on LUAD with K fixed at 3)
- Posterior predictive zero rates match observed zero rates per batch (e.g., 0.887 vs 0.892 for hematopoietic MARS-seq), while a dropout-free BUSseq-nzf fit underestimates them (0.874)
- Inferred size factors agree with established normalization factors (DESeq, TMM, library-size, deconvolution benchmarks)
- MCMC folders MCMC_sampling_K[K] and Inference_K[K] are created with converged posterior samples after the specified burn-in (e.g., 4000 iterations)

## Limitations

- Completely confounded designs admit no applicable method — BUSseq requires designs (completely randomized, reference panel, or chain-type) under which biological variability is identifiable from batch effects
- MCMC algorithms carry heavy computation load; runtimes reported on 4 Intel i7-6700HQ cores (e.g., ~12 minutes for 4 K values on the 450-cell demo, ~2 minutes per K for 500 iterations on LUAD)
- Clustering errors propagate: downstream analyses that assume correct cell-type labels can yield false positives and false negatives, unlike BUSseq's joint inference

## Evidence

- [methods] BUSseq_MCMC(ObservedData, n.celltypes, seed, n.cores, n.iterations, n.burnin, n.unchanged, n.output, working_dir, Drop_ind, fdr, hyper_pi, hyper_gamma0, hyper_gamma1, hyper_alpha, tau1sq, hyper_p,: "BUSseq_MCMC(ObservedData, n.celltypes, seed, n.cores, n.iterations, n.burnin, n.unchanged, n.output, working_dir, Drop_ind, fdr, hyper_pi, hyper_gamma0, hyper_gamma1, hyper_alpha, tau1sq, hyper_p,"
- [methods] To select the optimal K, we need to compare the BIC values for different Ks. In the vignettes, we enumerate K from 3 to 6.: "To select the optimal *K*, we need to compare the BIC values for different *K*s. In the vignettes, we enumerate *K* from 3 to 6."
- [discussion] most steps of the MCMC algorithm for BUSseq are parallelizable (Supplementary Note 12).: "most steps of the MCMC algorithm for BUSseq are parallelizable (Supplementary Note 12)."
- [readme] The compiling of C++ codes will only take a few seconds.: [evidence span withheld — anchor status: external_doc]
- [readme] BUSseq can integrate read count matrices obtained from different scRNA-seq platforms and allow cell types to be measured in some but not all of the batches: "BUSseq can integrate read count matrices obtained from different scRNA-seq platforms and allow cell types to be measured in some but not all of the batches as long as the experimental design fulfills"
