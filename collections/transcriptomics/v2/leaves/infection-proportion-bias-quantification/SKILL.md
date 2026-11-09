---
name: infection-proportion-bias-quantification
description: Use when analyzing scCRISPR-seq perturbation vs. control data where cluster
  proportions differ between groups, and you need to measure or validate how much
  fold-change estimates are contaminated by infected proportion bias.
when_to_use_negative:
- When cell cluster proportions are identical between perturbation and control groups
  — there is no infected proportion bias to quantify
- 'When the data violate scDecouple''s assumptions: non-Gaussian-mixture expression
  distributions (e.g. tumor tissues) or rare cell types completely absent from infection'
edam_operation: http://edamontology.org/operation_0334
edam_topics:
- http://edamontology.org/topic_3173
tools:
- name: scDecouple
  role: R package performing infected proportion bias quantification via maximum likelihood
    estimation with Gaussian mixture models and EM
  repo: https://github.com/MengQiuchen/scDecouple
- name: tidyverse
  role: Assembling per-parameter bias, d, and r values into a tidy table and generating
    ggplot2 regression plots
- name: R
  role: Environment for running scDecouple and computing Pearson correlations of bias
    against d and r
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bib/bbae011
    title: 'scDecouple: decoupling cellular response from infected proportion bias
      in scCRISPR-seq'
  source_content_hash: sha256:b3e3a592b392565e99f60a271610bae44f87249b5121ca95a380deb13a8c468a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MengQiuchen/scDecouple
  tools:
  - R
  - scDecouple
  - tidyverse
  edam_operation: http://edamontology.org/operation_0334
  edam_topics:
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# infection-proportion-bias-quantification

## Summary

Quantifies the infected proportion bias that distorts perturbation effect estimates in scCRISPR-seq, including validating its linear dependence on cluster distance d and ratio change r (Equations 14-15) in synthetic two-cluster scenarios.

## When to use

Use when analyzing scCRISPR-seq perturbation vs. control data where cluster proportions differ between groups, and you need to measure or validate how much fold-change estimates are contaminated by infected proportion bias.

## When NOT to use

- When cell cluster proportions are identical between perturbation and control groups — there is no infected proportion bias to quantify
- When the data violate scDecouple's assumptions: non-Gaussian-mixture expression distributions (e.g. tumor tissues) or rare cell types completely absent from infection

## Inputs

- Perturbation group cell-by-gene expression matrix Y (scCRISPR-seq)
- Control group cell-by-gene expression matrix Z (e.g. NT gRNA cells)
- Synthetic two-cluster perturbation/control matrices spanning a simulated (d, r) parameter grid

## Outputs

- Per-PC infected proportion bias quantification (from scDecouple step3)
- Tidy table of bias vs. d and r per parameter combination (bias_vs_dr_table.csv)
- ggplot2 figure of bias vs. d and bias vs. r with fitted linear regressions and Pearson correlations (bias_vs_dr_correlation.png)

## How to apply

Run the scDecouple R package on perturbation matrix Y and control matrix Z: preprocess (data_preprocessing), select multimodal PCs via pc_selection (using_rank=TRUE with top.sdev=30, top.multimodal=3 by default), then run scDecouple() to decouple cellular response from infection bias and quantify the bias per PC. For synthetic two-cluster validation, compute the bias term (lambda_1 - delta_1)(mu_1 - mu_2) — the product of the cluster-proportion difference and the mean-expression difference — across a simulated grid of cluster distance d and ratio change r, then confirm via Pearson correlation that bias varies linearly with both d and r as derived in Equations 14-15. Assemble per-parameter bias values with tidyverse/dplyr and visualize bias versus d and r with ggplot2 regression lines annotated with correlation coefficients.

## Related tools

- **scDecouple** (R package performing infected proportion bias quantification via maximum likelihood estimation with Gaussian mixture models and EM) — https://github.com/MengQiuchen/scDecouple
- **tidyverse** (Assembling per-parameter bias, d, and r values into a tidy table and generating ggplot2 regression plots)
- **R** (Environment for running scDecouple and computing Pearson correlations of bias against d and r)

## Examples

```
res.step2 <- pc_selection(Z.pca.mat = res.step1$Z.pca.results$x, Z.pca.sdev = res.step1$Z.pca.results$sdev, using_rank = TRUE); res.step3 <- scDecouple(Z.pca.mat = res.step1$Z.pca.results$x, Z.pca.rotation = res.step1$Z.pca.results$rotation, Y.norm.log.sub = res.step1$Y.norm.log.sub, select.pcs = res.step2)
```

## Evaluation signals

- Bias values show approximately linear trends with both d and r (high positive Pearson correlation), consistent with the paper's linear derivation in Equations 14-15
- The bias-vs-(d,r) table contains one row per simulated (d, r) combination with no missing bias values
- For the two-cluster case, the per-PC bias equals exactly the product (lambda_1 - delta_1)(mu_1 - mu_2)
- Selected PCs include those flagged as potentially having high infection bias by pc_selection (rank-based thresholds top.sdev=30, top.multimodal=3)

## Limitations

- GMM assumption may not hold in intricate systems or disease tissues such as tumors
- Assumption that the same cell clusters appear in NT and perturbation groups fails when rare cell types are completely absent from infection
- Similar cellular responses across cell clusters limits applicability to cells with moderate heterogeneity

## Evidence

- [abstract] We developed scDecouple to decouple true cellular response of perturbation from the influence of infected proportion bias.: "We developed scDecouple to decouple true cellular response of perturbation from the influence of infected proportion bias."
- [intro] The infection proportion bias refers to significant differences in the proportions of cell clusters between the perturbation and control groups.: "the data analysis may suffer from gRNA ‘infected proportion bias’. The infection proportion bias refers to the significant differences in the proportions of cell clusters between the perturbation"
- [results] Two-cluster bias factorizes as product of ratio change and cluster distance.: [evidence span withheld — anchor status: not_found]
- [methods] Step 2 identifies PCs which may have high infection bias.: "## step2: get PCs which may have high infection bias"
- [readme] README documents pc_selection rank-based thresholds.: "top.sdev` (optional): When using_rank=TRUE, this parameter sets the threshold of standard deviations of PCs. The default is 30."
