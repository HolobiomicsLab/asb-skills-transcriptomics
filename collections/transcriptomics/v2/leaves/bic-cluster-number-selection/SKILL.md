---
name: bic-cluster-number-selection
description: Use when fitting Gaussian mixture models to principal components of scCRISPR-seq
  control/perturbation expression data and the number of cell clusters K is unknown
  and must be chosen objectively.
when_to_use_negative:
- The PC being modeled is unimodal (fails the multimodality/dip-test screen of pc_selection),
  so a Gaussian mixture with K >= 2 is not warranted.
- Cell clusters are expected to be so heterogeneous that the GMM assumption and the
  assumption of similar cellular responses across clusters fail, e.g. intricate systems
  or disease tissues such as tumors.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0603
tools:
- name: scDecouple
  role: R package whose decoupling step fits per-PC GMMs and selects the shared cluster
    number K by BIC before running EM
  repo: https://github.com/MengQiuchen/scDecouple
- name: mixtools
  role: R library loaded to support Gaussian mixture model fitting within the scDecouple
    workflow
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bib/bbae011
    title: 'scDecouple: decoupling cellular response from infected proportion bias
      in scCRISPR-seq'
  source_content_hash: sha256:88fc12b3b4b574a5b99dd4336c96ac661c02355c1c123b1565fcd8b12dff653d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MengQiuchen/scDecouple
  tools:
  - R
  - mixtools
  - scDecouple
  - tidyverse
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0603
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# bic-cluster-number-selection

## Summary

Selects the number of mixture components for the shared-K Gaussian mixture models fitted per infection-biased principal component in scDecouple, using Bayesian Information Criterion (BIC) so that the control and perturbation groups are modeled with the same cluster number before EM estimation.

## When to use

Use when fitting Gaussian mixture models to principal components of scCRISPR-seq control/perturbation expression data and the number of cell clusters K is unknown and must be chosen objectively.

## When NOT to use

- The PC being modeled is unimodal (fails the multimodality/dip-test screen of pc_selection), so a Gaussian mixture with K >= 2 is not warranted.
- Cell clusters are expected to be so heterogeneous that the GMM assumption and the assumption of similar cellular responses across clusters fail, e.g. intricate systems or disease tissues such as tumors.

## Inputs

- Control-group PCA scores matrix (res.step1$Z.pca.results$x) from data_preprocessing
- Control-group PCA rotation (loadings) matrix (res.step1$Z.pca.results$rotation)
- Log-normalized perturbation-group expression subset (res.step1$Y.norm.log.sub), cells by genes
- Vector of selected infection-biased PCs (output of pc_selection, res.step2)
- Random seed for the EM algorithm (seed_, default 0)

## Outputs

- Per-PC fitted GMMs for control and perturbation groups with a shared cluster number K chosen by BIC
- Per-gRNA estimated cluster proportions (lambda) and cellular response coefficients (beta), returned as beta.PC.scDecouple in the step-3 result object
- Precision parameters (M) estimated by EM for each gRNA

## How to apply

After data_preprocessing (log-normalization, HVG detection) and pc_selection (using_rank = TRUE with default thresholds top.sdev = 30 and top.multimodal = 3 on dip statistics), pass the selected PCs to the scDecouple() function. Internally, for each selected PC, GMMs with differing component counts are fitted to the control (Z) and perturbation (Y) groups, and the shared cluster number K is chosen by BIC rather than fixed a priori; using one shared K across both groups is what allows cluster proportions (lambda) and cellular response coefficients (beta) to be compared and estimated per gRNA. Supply a random seed (default seed_ = 0) since GMM fitting and the subsequent EM iterations are stochastic. The rationale is that K must reflect the true multimodal structure of the expression distribution along each PC; a wrong K corrupts both the estimated infected proportion and the decoupled cellular response.

## Related tools

- **scDecouple** (R package whose decoupling step fits per-PC GMMs and selects the shared cluster number K by BIC before running EM) — https://github.com/MengQiuchen/scDecouple
- **mixtools** (R library loaded to support Gaussian mixture model fitting within the scDecouple workflow)

## Examples

```
res.step3 <- scDecouple(Z.pca.mat = res.step1$Z.pca.results$x, Z.pca.rotation = res.step1$Z.pca.results$rotation, Y.norm.log.sub = res.step1$Y.norm.log.sub, select.pcs = res.step2, seed_ = 0)
```

## Evaluation signals

- The same cluster number K is used for both the control-group and perturbation-group GMMs on each selected PC (shared-K requirement).
- BIC is the reported selection criterion, i.e. the chosen K maximizes the BIC among candidate mixture models per PC.
- The EM algorithm runs to maximum likelihood convergence, yielding lambda (cluster proportions), beta (cellular response coefficients) and M (precisions) for each gRNA.
- The returned step-3 object contains beta.PC.scDecouple for every PC listed in select.pcs, and the complete result list can be saved as scDecouple_results.rds.
- Re-running with the same seed_ (default 0) reproduces the same K selection and EM estimates.

## Limitations

- The GMM assumption (clusters as Gaussian components in PC space) may not hold in intricate systems or disease tissues such as tumors.
- The same-cell-clusters-between-NT-and-perturbation-groups assumption can fail when some rare cell types are completely absent from infection.
- The similar-cellular-responses-across-clusters assumption limits scDecouple to cells with moderate heterogeneity.

## Evidence

- [other] Shared-K BIC selection in the decoupling stage: [evidence span withheld — anchor status: not_found]
- [readme] GMM modeling and ML iteration over cluster proportions: "It models the distribution of gene expression profiles in perturbed cells and then iteratively finds the maximum likelihood of cell cluster proportions as well as the cellular response for each gRNA."
- [intro] Maximum likelihood estimation rationale: "Here, we developed scDecouple as a solution to solve this problem by employing maximum likelihood estimation."
- [other] EM estimates lambda, beta and M: [evidence span withheld — anchor status: not_found]
