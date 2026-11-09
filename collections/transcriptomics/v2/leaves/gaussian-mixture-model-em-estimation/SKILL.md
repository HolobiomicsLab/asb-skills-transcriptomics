---
name: gaussian-mixture-model-em-estimation
description: Use when scCRISPR-seq perturbation vs control groups differ in cluster
  proportions (infected proportion bias) and you need per-gRNA cellular responses
  decoupled from that bias on selected infection-biased PCs.
when_to_use_negative:
- Cell populations where the GMM assumption may fail, e.g. intricate systems or disease
  tissues such as tumors
- Extreme cases where some rare cell types are completely absent from infection, so
  cluster sets differ between NT and perturbation groups
- Samples with strong heterogeneity rather than moderate heterogeneity, since similar
  cellular responses across cell clusters is assumed
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scDecouple
  role: R package implementing the GMM-EM decoupling of cellular response from infected
    proportion bias per gRNA
  repo: https://github.com/MengQiuchen/scDecouple
- name: mixtools
  role: R library for Gaussian mixture model fitting loaded in the scDecouple workflow
- name: devtools
  role: Used to install scDecouple from GitHub with build_vignettes=TRUE
provenance:
  source_task_ids:
  - task_002
  - task_005
  - task_006
  source_papers:
  - doi: 10.1093/bib/bbae011
    title: 'scDecouple: decoupling cellular response from infected proportion bias
      in scCRISPR-seq'
  source_content_hash: sha256:b6581679e05f869c927194242f123ffb8388a965bcc801ef05af09f809294a3b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MengQiuchen/scDecouple
  tools:
  - R
  - Seurat
  - devtools
  - diptest
  - enrichR
  - mixtools
  - scDecouple
  - tidyverse
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 14
---

# gaussian-mixture-model-em-estimation

## Summary

Fit shared-K Gaussian mixture models to the control and perturbation groups in PC space and run EM to estimate per-gRNA cluster proportions (lambda), cellular response coefficients (beta), and precisions, decoupling true cellular response from infected proportion bias in scCRISPR-seq.

## When to use

Use when scCRISPR-seq perturbation vs control groups differ in cluster proportions (infected proportion bias) and you need per-gRNA cellular responses decoupled from that bias on selected infection-biased PCs.

## When NOT to use

- Cell populations where the GMM assumption may fail, e.g. intricate systems or disease tissues such as tumors
- Extreme cases where some rare cell types are completely absent from infection, so cluster sets differ between NT and perturbation groups
- Samples with strong heterogeneity rather than moderate heterogeneity, since similar cellular responses across cell clusters is assumed

## Inputs

- Z.pca.mat: control-group PCA scores (Z.pca.results$x from data_preprocessing)
- Z.pca.rotation: control-group PCA loadings (Z.pca.results$rotation)
- Y.norm.log.sub: perturbation-group log-normalized, HVG-subset cell-by-gene matrix
- select.pcs: infection-biased selected PCs from pc_selection
- seed_ (default 0): random seed for the EM algorithm

## Outputs

- beta.PC.scDecouple: per-PC cellular response coefficients per gRNA
- Per-gRNA cluster proportions (lambda) and precisions (M) from the shared-K GMMs
- Complete scDecouple result list (e.g. saved as scDecouple_results.rds)

## How to apply

First run data_preprocessing to get log-normalized matrices and highly variable genes, then pc_selection on the control-group PCA (using_rank=TRUE, top.sdev=30, top.multimodal=3) to obtain infection-biased PCs. Call scDecouple() with Z.pca.mat, Z.pca.rotation, Y.norm.log.sub and select.pcs; it internally fits Gaussian mixture models per selected PC with the shared cluster number K chosen by BIC (e.g. K=2). Run the EM algorithm (seed_=0 for reproducibility) to iteratively estimate cluster proportions (lambda), cellular response coefficients (beta) and precisions (M) for each gRNA until maximum likelihood convergence. Extract beta.PC.scDecouple for downstream gene ranking, and compare against the naive observed-change (OC) estimate via MAE on PC 1 responses to confirm improvement.

## Related tools

- **scDecouple** (R package implementing the GMM-EM decoupling of cellular response from infected proportion bias per gRNA) — https://github.com/MengQiuchen/scDecouple
- **mixtools** (R library for Gaussian mixture model fitting loaded in the scDecouple workflow)
- **devtools** (Used to install scDecouple from GitHub with build_vignettes=TRUE)

## Examples

```
res.step3 <- scDecouple(Z.pca.mat = res.step1$Z.pca.results$x, Z.pca.rotation = res.step1$Z.pca.results$rotation, Y.norm.log.sub = res.step1$Y.norm.log.sub, select.pcs = res.step2, seed_ = 0)
```

## Evaluation signals

- EM iteration converges by maximum likelihood for the two shared-K GMMs (control and perturbation) on each selected PC
- Shared cluster number K across control and perturbation GMMs is consistent with BIC selection (e.g. K=2)
- Per-PC cellular response beta.PC.scDecouple is extractable from the returned step-3 result object
- MAE of scDecouple estimated cellular responses on PC 1 is lower than the naive observed-change (OC) estimate, with the greatest advantage when infected cells show substantial cluster ratio change
- Downstream gene rankings (e.g. CD69 and RELA positions) improve over OC, consistent with reported average ranking gains (280 and 125 on Jurkat)

## Limitations

- GMM assumption may not hold in intricate systems or disease tissues such as tumors
- Assumes the same cell clusters exist between NT and perturbation groups; fails when rare cell types are completely absent from infection
- Assumes similar cellular responses across cell clusters, limiting use to cells with moderate heterogeneity

## Evidence

- [methods] EM on shared-K GMMs: [evidence span withheld — anchor status: not_found]
- [intro] GMM depicts distributions: "scDecouple leverages a Gaussian mixture model to depict the distribution of gene"
- [methods] workflow call signature: "res.step3 <- scDecouple(Z.pca.mat = res.step1$Z.pca.results$x, Z.pca.rotation = res.step1$Z.pca.results$rotation, Y.norm.log.sub = res.step1$Y.norm.log.sub, select.pcs = res.step2)"
- [readme] demo one-stop call: "res <- run_scDecouple(Y, Z)"
- [results] MAE advantage over OC: [evidence span withheld — anchor status: not_found]
