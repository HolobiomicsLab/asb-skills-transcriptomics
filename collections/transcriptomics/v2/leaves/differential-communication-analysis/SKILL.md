---
name: differential-communication-analysis
description: Use when testing whether cell-cell communication strength mediated by
  specific candidate eLR pairs depends on a developmental process, by comparing interaction
  scores between a perturbed condition (e.g., ZGA-inhibited embryos) and unperturbed
  time-point conditions.
when_to_use_negative:
- Communication inference between well-defined cell types rather than perturbation-vs-control
  differential testing — the article notes CellPhoneDB and CellChat are designed for
  communication between given cell types
- Studying maternal-embryo interactions — the TimeTalk autocrine assumption in eLR
  screening precludes this
- Communication not mediated by ligand-receptor interactions (e.g., extracellular-vesicle-mediated)
  — the current TimeTalk framework does not account for other communication types
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: TimeTalk
  role: R framework providing the analysis scripts used to compute interaction scores
    and classify differentially altered eLR pairs
  repo: https://github.com/ChengLiLab/TimeTalk
- name: R
  role: Environment for all statistical analyses (version 3.6.3 in the reference study),
    including delta computation and thresholding
  repo: https://github.com/ChengLiLab/TimeTalk
- name: Seurat
  role: Preprocessing of the merged expression matrix (version 3.2.3; e.g., CreateSeuratObject
    with min.features = 2000) prior to pseudo-bulk aggregation
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s42003-023-05283-2
    title: TimeTalk uses single-cell RNA-seq datasets to decipher cell-cell communication
      during early embryo development
  source_content_hash: sha256:119d3b856f22e4db258ea09cb60c262a98202a87d569038e2132cd523886211e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ChengLiLab/TimeTalk
  tools:
  - R
  - Seurat
  - TimeTalk
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 4
---

# differential-communication-analysis

## Summary

Quantify how perturbation of a developmental process (e.g., ZGA inhibition by alpha-amanitin) changes the interaction score of embryo-related ligand-receptor pairs (eLRs) across matched developmental stages, using TimeTalk's R analysis scripts on pseudo-bulk per-condition expression.

## When to use

Use when testing whether cell-cell communication strength mediated by specific candidate eLR pairs depends on a developmental process, by comparing interaction scores between a perturbed condition (e.g., ZGA-inhibited embryos) and unperturbed time-point conditions.

## When NOT to use

- Communication inference between well-defined cell types rather than perturbation-vs-control differential testing — the article notes CellPhoneDB and CellChat are designed for communication between given cell types
- Studying maternal-embryo interactions — the TimeTalk autocrine assumption in eLR screening precludes this
- Communication not mediated by ligand-receptor interactions (e.g., extracellular-vesicle-mediated) — the current TimeTalk framework does not account for other communication types

## Inputs

- Perturbation bulk RNA-seq expression matrix (e.g., GEO GSE71434, alpha-amanitin-treated embryos)
- Unperturbed pre-implantation time-course scRNA-seq expression matrices (early 2-cell, late 2-cell)
- Curated mouse ligand-receptor pair list (e.g., CellTalkDB 2034 mouse pairs / Mouse-2020-Shao-LR-pairs.txt)
- Candidate eLR set from prior TimeTalk eLR identification

## Outputs

- Per-eLR interaction scores under each condition (early 2-cell, late 2-cell, ZGA-inhibited)
- Delta log-ratio value per eLR pair (perturbed vs unperturbed)
- Classified set of significantly altered eLR pairs (reference result: 56 pairs)
- Summary figure reproducing Fig. 3e / Supplementary Figs. 12-13

## How to apply

Download the perturbation RNA-seq dataset (e.g., GSE71434 alpha-amanitin-treated embryos) plus the unperturbed time-course scRNA-seq, and normalize the expression matrices (quantile normalization via preprocessCore 'normalize.quantiles'). Compute pseudo-bulk expression for an average single cell per condition (early 2-cell, late 2-cell, treated), then use TimeTalk's R analysis scripts to compute an interaction score for each eLR pair under each condition. Derive a delta value per eLR pair with the defined log-ratio formula comparing interaction scores between ZGA-inhibited and unperturbed 2-cell conditions. Apply the delta threshold to classify significantly altered pairs — the reference analysis recovered exactly 56 altered eLR pairs, shown in Fig. 3e and Supplementary Figs. 12-13. Run all statistical analyses in R (the authors used version 3.6.3) and, because results are parameter-sensitive, test multiple parameter combinations for dependable results.

## Related tools

- **TimeTalk** (R framework providing the analysis scripts used to compute interaction scores and classify differentially altered eLR pairs) — https://github.com/ChengLiLab/TimeTalk
- **R** (Environment for all statistical analyses (version 3.6.3 in the reference study), including delta computation and thresholding) — https://github.com/ChengLiLab/TimeTalk
- **Seurat** (Preprocessing of the merged expression matrix (version 3.2.3; e.g., CreateSeuratObject with min.features = 2000) prior to pseudo-bulk aggregation)

## Examples

```
tmp.res <- TimeTalk.result %>% filter(category == "PASS") %>% pull(LR) %>% unique()
```

## Evaluation signals

- The recovered altered-pair set contains exactly 56 eLR pairs, matching the manuscript's ZGA-inhibition analysis (Fig. 3e)
- Interaction scores are computed on pseudo-bulk per-condition expression for all three conditions (early 2-cell, late 2-cell, alpha-amanitin-treated)
- Every candidate eLR pair receives a defined delta log-ratio value before thresholding
- Reproduced Fig. 3e and Supplementary Figs. 12-13 visually match the published altered-pair summary
- All analyses run reproducibly in R 3.6.3 with quantile-normalized (preprocessCore) expression matrices

## Limitations

- The Granger causality conclusion is affected by the choice of the winsz parameter (Supplementary Figs. 20-21); users are advised to test various parameter combinations
- The autocrine hypothesis behind eLR screening precludes analysis of maternal-embryo interactions during early development
- The current TimeTalk framework covers only ligand-receptor-mediated communication; other communication types (e.g., EV-mediated) are not accounted for

## Evidence

- [discussion] ZGA inhibition RNA-seq data were obtained from a study by ref. 30.: "ZGA inhibition RNA-seq data were obtained from a study by ref. 30."
- [discussion] Custom R scripts were used for the pre-implantation scRNA-seq data to calculate the pseudo-bulk expression for an average single cell in each stage.: "Custom R scripts were used for the pre-implantation scRNA-seq data to calculate the pseudo-bulk expression for an average single cell in each stage."
- [discussion] Quantile normalization was performed with the function “normalize.quantiles” from R package preprocessCore (version 1.48.0).: "Quantile normalization was performed with the function “normalize.quantiles” from R package preprocessCore (version 1.48.0)."
- [methods] devtools::install_github("ChengLiLab/TimeTalk",ref="main"): "devtools::install_github("ChengLiLab/TimeTalk",ref="main")"
- [readme] we advise users to conduct additional data analysis and experiment with various parameter combinations to achieve more dependable results: "we advise users to conduct additional data analysis and experiment with various parameter combinations to achieve more dependable results"
