---
name: principal-component-multimodality-selection
description: Use when running scDecouple on perturbation matrix Y and control matrix
  Z from scCRISPR-seq (e.g. CROP-seq) data and you must pick which control-group PCs
  carry infection bias before the Gaussian-mixture decoupling step.
when_to_use_negative:
- When the perturbation and control groups are drawn from a single homogeneous cluster
  (no cluster-proportion differences between groups), since multimodality signals
  of infection bias would be absent.
- When some rare cell types are completely absent from infection, a case the article
  explicitly flags as violating scDecouple's assumptions.
- When the input is not a cell-by-gene scCRISPR-seq pair (perturbation Y and control
  Z) that has been log-normalized with highly variable genes detected, e.g. bulk RNA-seq
  or already-aggregated feature tables.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: scDecouple
  role: R package whose pc_selection() function performs rank- and dip-test-based
    selection of infection-biased PCs from the control-group PCA
  repo: https://github.com/MengQiuchen/scDecouple
- name: diptest
  role: R library used for the dip statistic assessing PC score multimodality (top.multimodal=3)
- name: Seurat
  role: R library loaded for single-cell handling in the scDecouple workflow preprocessing
    of the control matrix Z
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bib/bbae011
    title: 'scDecouple: decoupling cellular response from infected proportion bias
      in scCRISPR-seq'
  source_content_hash: sha256:74ca113f588e7f5b1a2179cf6d5dd714e6728d8fcc847fb8b8399f0fe28598fd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MengQiuchen/scDecouple
  tools:
  - R
  - Seurat
  - diptest
  - enrichR
  - mixtools
  - scDecouple
  - tidyverse
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# principal-component-multimodality-selection

## Summary

Select candidate infection-biased PCs from the control-group PCA in scCRISPR-seq data by combining standard-deviation rank with dip-test multimodality, so that only PCs plausibly carrying gRNA infected proportion bias are passed to the scDecouple EM decoupling step.

## When to use

Use when running scDecouple on perturbation matrix Y and control matrix Z from scCRISPR-seq (e.g. CROP-seq) data and you must pick which control-group PCs carry infection bias before the Gaussian-mixture decoupling step.

## When NOT to use

- When the perturbation and control groups are drawn from a single homogeneous cluster (no cluster-proportion differences between groups), since multimodality signals of infection bias would be absent.
- When some rare cell types are completely absent from infection, a case the article explicitly flags as violating scDecouple's assumptions.
- When the input is not a cell-by-gene scCRISPR-seq pair (perturbation Y and control Z) that has been log-normalized with highly variable genes detected, e.g. bulk RNA-seq or already-aggregated feature tables.

## Inputs

- Z.pca.mat: PCA score matrix of the log-normalized control group Z (res.step1$Z.pca.results$x)
- Z.pca.sdev: standard deviations of the control-group PCs (res.step1$Z.pca.results$sdev)
- using_rank logical flag; optional thresholds top.sdev (default 30), top.multimodal (default 3), or using_rank=FALSE with th.sdev=1 and th.multimodal=0.0125

## Outputs

- res.step2 / select.pcs: vector of selected infection-biased PCs passed to the scDecouple() decoupling step
- PC variance and multimodality visualizations (PC plots, PC variance and multimodality plots)

## How to apply

Run PCA on the log-normalized control group Z (from data_preprocessing) and call pc_selection on Z.pca.results$x and Z.pca.results$sdev. Set using_rank=TRUE so PCs are screened by standard-deviation rank (top.sdev=30 keeps the top-30 spread PCs) and by dip-test multimodality (top.multimodal=3 keeps the 3 most multimodal PCs), yielding the PCs flagged as potentially infection-biased. The rationale is that infected proportion bias appears as cluster-proportion differences, which manifest as multimodal PC score distributions; restricting to these candidates makes the subsequent K=2 Gaussian mixture model with EM (seed_=0) well-posed. If you prefer absolute thresholds over rank, set using_rank=FALSE with th.sdev=1 and th.multimodal=0.0125.

## Related tools

- **scDecouple** (R package whose pc_selection() function performs rank- and dip-test-based selection of infection-biased PCs from the control-group PCA) — https://github.com/MengQiuchen/scDecouple
- **diptest** (R library used for the dip statistic assessing PC score multimodality (top.multimodal=3))
- **Seurat** (R library loaded for single-cell handling in the scDecouple workflow preprocessing of the control matrix Z)

## Examples

```
res.step2 <- pc_selection(Z.pca.mat = res.step1$Z.pca.results$x, Z.pca.sdev = res.step1$Z.pca.results$sdev, using_rank = TRUE)
```

## Evaluation signals

- res.step2 (select.pcs) contains only PCs that satisfy the rank threshold (top.sdev=30) or the dip-test threshold (top.multimodal=3); on the Jurkat CROP-seq demo this is three infection-biased PCs.
- Downstream scDecouple() must receive the exact res.step2 object as select.pcs and the matching res.step1$Z.pca.results$x and $rotation, or the EM/GMM step is invalid.
- The subsequent K=2 Gaussian mixture with EM (seed_=0) should converge and produce beta.PC.scDecouple coefficients that decouple cellular response from infection bias on the selected PCs.
- If using_rank=FALSE is chosen, thresholds must be the documented th.sdev=1 and th.multimodal=0.0125 rather than the rank-based ones.
- Selected PCs should show multimodal score distributions consistent with cluster-proportion (infected proportion bias) differences between perturbation and control groups.

## Limitations

- The dip-test/GMM framework assumes a Gaussian mixture of cell clusters; the article notes this 'might not remain in intricate systems or disease tissues, such as tumors'.
- Assumes the same cell clusters exist in NT and perturbation groups; fails when rare cell types are completely absent from infection.
- Assumes similar cellular responses across cell clusters, limiting scDecouple to cells with moderate heterogeneity.
- PC selection is heuristic (rank-based top.sdev=30 / top.multimodal=3 or absolute th.sdev=1 / th.multimodal=0.0125); an inappropriate threshold can drop true infection-biased PCs or admit noise PCs.

## Evidence

- [methods] Rank/dip-test PC selection parameters: "res.step2 <- pc_selection(Z.pca.mat = res.step1$Z.pca.results$x, Z.pca.sdev = res.step1$Z.pca.results$sdev, using_rank = TRUE)"
- [methods] PC selection purpose in the workflow: "## step2: get PCs which may have high infection bias"
- [readme] Default thresholds documented in README: "4. `top.sdev` (optional): When using_rank=TRUE, this parameter sets the threshold of standard deviations of PCs. The default is 30."
- [readme] Multimodality threshold: "5. `top.multimodal` (optional): When using_rank=TRUE, this parameter sets the threshold of dip statistics. The default is 3."
- [readme] Alternative absolute-threshold mode: "6. `th.sdev` (optional): When using_rank=FALSE, this parameter sets the threshold of standard deviations of PCs. The default is 1."
