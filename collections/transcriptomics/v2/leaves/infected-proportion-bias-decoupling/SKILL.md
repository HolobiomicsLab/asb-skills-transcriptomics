---
name: infected-proportion-bias-decoupling
description: Use when analyzing scCRISPR-seq data where perturbation (Y) and control
  (Z) groups show differing cell-cluster proportions, so naive fold-change / observed-change
  (OC) estimates of perturbation effect are contaminated by infected proportion bias
  and the true cellular response must be recovered.
when_to_use_negative:
- Data from intricate systems or disease tissues such as tumors, where the GMM assumption
  on expression distributions may not hold
- Extreme cases where some rare cell types are completely absent from infection, violating
  the shared-cluster assumption between NT and perturbation groups
- Samples with strong heterogeneity, since the similar-cellular-response assumption
  limits scDecouple to cells with moderate heterogeneity
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scDecouple
  role: Core R package implementing the GMM-EM decoupling of cellular response from
    infected proportion bias (data_preprocessing, pc_selection, scDecouple, downstream_analysis,
    run_scDecouple)
  repo: https://github.com/MengQiuchen/scDecouple
- name: mixtools
  role: Provides the EM algorithm underlying the Gaussian mixture model fitting for
    cell cluster proportions and response
- name: diptest
  role: Dip test statistic used in PC selection to flag multimodal PCs with potential
    infection bias (top.multimodal threshold)
- name: tidyverse
  role: R data-manipulation library loaded in the scDecouple workflow and demo
- name: devtools
  role: Used to install the developmental scDecouple package from GitHub with build_vignettes
    = TRUE
  repo: https://github.com/MengQiuchen/scDecouple
provenance:
  source_task_ids:
  - task_003
  - task_005
  source_papers:
  - doi: 10.1093/bib/bbae011
    title: 'scDecouple: decoupling cellular response from infected proportion bias
      in scCRISPR-seq'
  source_content_hash: sha256:af07ef5e3d6cac6335a64f92010ae1ca6b9509dfc07dee28a90acd1d8d16f010
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
triage_status: promoted
yes_rate: 0.7058823529411765
n_claims: 17
---

# infected-proportion-bias-decoupling

## Summary

A maximum-likelihood decoupling method (scDecouple) that separates the true cellular response to CRISPR perturbation from 'infected proportion bias' — the distortion caused when cell-cluster proportions differ between the perturbation group Y and control group Z in scCRISPR-seq. It uses a Gaussian mixture model with EM, applied on principal components of the control group, outperforming naive observed-change/fold-change estimates.

## When to use

Use when analyzing scCRISPR-seq data where perturbation (Y) and control (Z) groups show differing cell-cluster proportions, so naive fold-change / observed-change (OC) estimates of perturbation effect are contaminated by infected proportion bias and the true cellular response must be recovered.

## When NOT to use

- Data from intricate systems or disease tissues such as tumors, where the GMM assumption on expression distributions may not hold
- Extreme cases where some rare cell types are completely absent from infection, violating the shared-cluster assumption between NT and perturbation groups
- Samples with strong heterogeneity, since the similar-cellular-response assumption limits scDecouple to cells with moderate heterogeneity

## Inputs

- Y: perturbation group cell-by-gene expression matrix (scCRISPR-seq counts)
- Z: control group (unperturbed or NT gRNA cells) cell-by-gene expression matrix
- Optional parameters: using_rank (default TRUE), top.sdev (default 30), top.multimodal (default 3), th.sdev (default 1), th.multimodal (default 0.0125), degenes.num (default 1500), seed_ (default 0)

## Outputs

- Per-gRNA cellular response estimates on selected PCs (beta.PC.scDecouple)
- Selected PCs with potential high infection bias (pc_selection result)
- Infected proportion bias quantification and estimated cluster proportions
- Gene ranking and GO pathway enrichment results (downstream_analysis)
- Visualizations: PC plots, PC variance/multimodality plots, cellular response heatmaps

## How to apply

Run the scDecouple pipeline on a cell-by-gene perturbation matrix Y and control matrix Z. First call data_preprocessing(Y, Z) to obtain log-normalized expression and highly variable genes; then pc_selection on the control-group PCA (using_rank = TRUE, top.sdev = 30, top.multimodal dip-statistic threshold = 3) to pick PCs likely carrying infection bias; then apply the GMM-EM scDecouple step (seed_ = 0 for reproducibility) to iteratively estimate maximum-likelihood cluster proportions and per-gRNA cellular response (beta.PC.scDecouple) on selected PCs, using mixtools for EM and diptest for multimodality. Optionally use the one-shot run_scDecouple(Y, Z) wrapper, then downstream_analysis for gene ranking and GO enrichment (degenes.num default 1500). The rationale: cluster-proportion differences shift mixture weights, so modeling the perturbed-cell distribution as a Gaussian mixture and re-estimating proportions recovers the response that FC/OC cannot. In simulation benchmarks, judge correctness by correlation (OC) of estimated vs ground-truth response on the response-bearing PC 1 — naive OC loss grows with cluster distance d and ratio change r while scDecouple stays accurate; on K562/RPE1 mixed data, compare MAE across 168 shared editing loci.

## Related tools

- **scDecouple** (Core R package implementing the GMM-EM decoupling of cellular response from infected proportion bias (data_preprocessing, pc_selection, scDecouple, downstream_analysis, run_scDecouple)) — https://github.com/MengQiuchen/scDecouple
- **mixtools** (Provides the EM algorithm underlying the Gaussian mixture model fitting for cell cluster proportions and response)
- **diptest** (Dip test statistic used in PC selection to flag multimodal PCs with potential infection bias (top.multimodal threshold))
- **tidyverse** (R data-manipulation library loaded in the scDecouple workflow and demo)
- **devtools** (Used to install the developmental scDecouple package from GitHub with build_vignettes = TRUE) — https://github.com/MengQiuchen/scDecouple

## Examples

```
library(scDecouple); res <- run_scDecouple(Y, Z)
```

## Evaluation signals

- In two-cluster synthetic simulations sweeping cluster distance d (r = 0.1) and ratio change r (d = 3), naive OC correlation degrades with increasing infected proportion bias while scDecouple OC on PC 1 stays accurate across 100 replicates per setting
- On simulated K562/RPE1 two-cluster data, scDecouple MAE across the 168 shared editing loci on PC 1 responses is lower than naive OC, with the largest advantage under substantial cluster ratio change; the advantage persists under dropout (batch-effect) noise
- PC selection returns only PCs whose control-group standard deviation exceeds top.sdev = 30 (or dip statistic above 3) when using_rank = TRUE
- Stepwise and one-shot runs agree: res <- run_scDecouple(Y, Z) reproduces the data_preprocessing -> pc_selection -> scDecouple chain (same seed_ = 0)
- Estimated beta.PC.scDecouple can be mapped back through Z.pca.rotation in downstream_analysis to produce gene rankings and GO enrichment without errors

## Limitations

- The GMM assumption holds well in certain cell lines and tissues, but might not remain valid in intricate systems or disease tissues, such as tumors
- The assumption of the same cell clusters between NT and perturbation groups may fail when some rare cell types are completely absent from infection
- The similar cellular responses across cell clusters assumption limits scDecouple to cells with moderate heterogeneity

## Evidence

- [abstract] We developed scDecouple to decouple true cellular response of perturbation from the influence of infected proportion bias.: "We developed scDecouple to decouple true cellular response of perturbation from the influence of infected proportion bias."
- [intro] GMM-EM method basis: "scDecouple leverages a Gaussian mixture model to depict the distribution of gene"
- [methods] Stepwise workflow: "res.step3 <- scDecouple(Z.pca.mat = res.step1$Z.pca.results$x, Z.pca.rotation = res.step1$Z.pca.results$rotation, Y.norm.log.sub = res.step1$Y.norm.log.sub, select.pcs = res.step2)"
- [results] Simulation validation behavior: [evidence span withheld — anchor status: not_found]
- [readme] One-shot invocation: "result <- run_scDecouple(Y, Z)"
- [discussion] GMM limitation: "The first assumption holds well in certain cell lines and tissues, but it might not remain in intricate systems or disease tissues, such as tumors"
