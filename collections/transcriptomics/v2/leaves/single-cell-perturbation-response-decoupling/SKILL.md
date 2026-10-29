---
name: single-cell-perturbation-response-decoupling
description: Use when analyzing scCRISPR-seq data (cell-by-gene matrices Y = perturbation
  group, Z = control group) where differing infected-cell cluster proportions between
  perturbation and control groups make fold-change perturbation effects unreliable
  and per-gRNA cellular response must be estimated.
when_to_use_negative:
- Disease tissues or intricate systems such as tumors where the Gaussian mixture assumption
  for expression distributions may not hold
- Extreme cases where some rare cell types are completely absent from infection, so
  the same-cluster assumption between NT and perturbation groups fails
- Datasets with strong heterogeneity where cellular responses differ across cell clusters
  — scDecouple assumes similar responses across clusters and suits cells with moderate
  heterogeneity
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scDecouple
  role: Core R package implementing the GMM-EM decoupling of cellular response from
    infected proportion bias
  repo: https://github.com/MengQiuchen/scDecouple
- name: mixtools
  role: R library loaded to support fitting the Gaussian mixture models per selected
    PC
- name: tidyverse
  role: R library used for data handling in the scDecouple workflow
- name: devtools
  role: Used to install the developmental scDecouple package from GitHub with build_vignettes
    = TRUE
  repo: https://github.com/MengQiuchen/scDecouple
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bib/bbae011
    title: 'scDecouple: decoupling cellular response from infected proportion bias
      in scCRISPR-seq'
  source_content_hash: sha256:b41fb689c0dac6df4ac3c58ac52b62f47acaaf5637f1209041a1b14718d6e448
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MengQiuchen/scDecouple
  tools:
  - R
  - devtools
  - mixtools
  - scDecouple
  - tidyverse
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# single-cell-perturbation-response-decoupling

## Summary

Applies scDecouple's GMM-EM decoupling core to separate true cellular response of each gRNA from infected proportion bias in scCRISPR-seq, by fitting shared-K Gaussian mixture models to control and perturbation groups in selected PC space and running maximum-likelihood estimation of cluster proportions (lambda), response coefficients (beta), and precisions (M).

## When to use

Use when analyzing scCRISPR-seq data (cell-by-gene matrices Y = perturbation group, Z = control group) where differing infected-cell cluster proportions between perturbation and control groups make fold-change perturbation effects unreliable and per-gRNA cellular response must be estimated.

## When NOT to use

- Disease tissues or intricate systems such as tumors where the Gaussian mixture assumption for expression distributions may not hold
- Extreme cases where some rare cell types are completely absent from infection, so the same-cluster assumption between NT and perturbation groups fails
- Datasets with strong heterogeneity where cellular responses differ across cell clusters — scDecouple assumes similar responses across clusters and suits cells with moderate heterogeneity

## Inputs

- Y: perturbation-group cell-by-gene matrix
- Z: control-group cell-by-gene matrix (unperturbed or NT gRNA cells)
- Optional parameters: using_rank (default TRUE), top.sdev = 30, top.multimodal = 3, th.sdev = 1, th.multimodal = 0.0125, degenes.num = 1500, seed_ = 0
- Demo data object test_data (data(test_data)) providing Y and Z

## Outputs

- Per-PC cellular response coefficients beta.PC.scDecouple from res.step3
- Estimated cluster proportions (lambda), response coefficients (beta), and precisions (M) per gRNA per selected PC
- Selected PCs with high infection bias (res.step2)
- Downstream gene ranking and GO pathway enrichment (res.step4, default 1500 DE genes)

## How to apply

Run the scDecouple step-by-step pipeline in R: first data_preprocessing(Y, Z) to obtain log-normalized data, highly variable genes, and the control-group PCA (Z.pca.results$x, Z.pca.results$rotation, Y.norm.log.sub, Z.norm.log.sub); second pc_selection on the control PCA with using_rank = TRUE and default thresholds top.sdev = 30 and top.multimodal = 3 (dip statistics) to identify PCs with high infection bias; third call scDecouple(Z.pca.mat, Z.pca.rotation, Y.norm.log.sub, select.pcs, seed_ = 0), which internally fits Gaussian mixture models per selected PC with the shared cluster number chosen by BIC. The core decoupling runs an EM algorithm over the two shared-K GMMs (control and perturbation groups) to iteratively estimate cluster proportions (lambda), cellular response coefficients (beta), and precisions (M) for each gRNA until maximum-likelihood convergence. Extract beta.PC.scDecouple from the step-3 result object and optionally pass it with Z.pca.rotation to downstream_analysis for gene ranking and GO enrichment (default degenes.num = 1500).

## Related tools

- **scDecouple** (Core R package implementing the GMM-EM decoupling of cellular response from infected proportion bias) — https://github.com/MengQiuchen/scDecouple
- **mixtools** (R library loaded to support fitting the Gaussian mixture models per selected PC)
- **tidyverse** (R library used for data handling in the scDecouple workflow)
- **devtools** (Used to install the developmental scDecouple package from GitHub with build_vignettes = TRUE) — https://github.com/MengQiuchen/scDecouple

## Examples

```
res.step3 <- scDecouple(Z.pca.mat = res.step1$Z.pca.results$x, Z.pca.rotation = res.step1$Z.pca.results$rotation, Y.norm.log.sub = res.step1$Y.norm.log.sub, select.pcs = res.step2, seed_ = 0)
```

## Evaluation signals

- EM iteration reaches maximum-likelihood convergence for lambda, beta, and M estimates
- Shared cluster number K for control and perturbation GMMs is selected by BIC
- res.step3 result object contains a non-empty beta.PC.scDecouple entry for each selected PC
- Selected PCs from pc_selection are limited to those with sdev <= 30 and significant multimodality (dip statistic) under using_rank = TRUE defaults
- Downstream gene ranking and GO enrichment (degenes.num = 1500) produce biologically perturbation-related genes as the article reports for real scCRISPR-seq data

## Limitations

- The Gaussian mixture assumption on expression distributions holds well in certain cell lines and tissues but may not remain in intricate systems or disease tissues such as tumors
- The assumption of same cell clusters between NT and perturbation groups might not hold when some rare cell types are completely absent from infection
- The similar cellular responses across cell clusters assumption limits scDecouple to cells with moderate heterogeneity
- Infected proportion bias distorts genuine cellular response, so plain fold-change estimates should not be substituted for decoupled responses

## Evidence

- [abstract] abstract: scDecouple decoupling: "We developed scDecouple to decouple true cellular response of perturbation from the influence of infected proportion bias."
- [intro] intro: GMM/EM: "Here, we developed scDecouple as a solution to solve this problem by employing maximum likelihood estimation. scDecouple leverages a Gaussian mixture model to depict the distribution of gene"
- [methods] methods: EM parameter estimation: [evidence span withheld — anchor status: not_found]
- [readme] readme: step-by-step pipeline: "res.step3 <- scDecouple(Z.pca.mat = res.step1$Z.pca.results$x, Z.pca.rotation = res.step1$Z.pca.results$rotation, Y.norm.log.sub = res.step1$Y.norm.log.sub, select.pcs = res.step2)"
- [readme] readme: PC selection thresholds: "`top.sdev` (optional): When using_rank=TRUE, this parameter sets the threshold of standard deviations of PCs. The default is 30."
- [discussion] discussion: GMM limitation: "The first assumption holds well in certain cell lines and tissues, but it might not remain in intricate systems or disease tissues, such as tumors"
