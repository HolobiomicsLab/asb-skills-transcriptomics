---
name: roc-auc-curve-evaluation
description: Use when comparing ranked drug/compound predictions from two or more
  pipelines (e.g., ASGARD multi-cluster score vs. CLUE, DrInsight, Alakwaa's, Guo's)
  against a known positive-drug ground truth and the comparison must be quantified
  as ROC/AUC with statistical significance testing.
when_to_use_negative:
- When no ground-truth positives exist — e.g., exploratory drug recommendations without
  a Supplementary-Data-4-style advanced-trial drug list to define positive cases
- When comparing methods by ranking metrics other than binary classification performance
  (e.g., drug-response correlation validated via TRANSACT with TNBC patient samples),
  where ROC/AUC against trial-status positives is not the appropriate yardstick
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_2269
tools:
- name: pROC
  role: R package used to compute ROC curves and AUCs for every pipeline (ASGARD,
    CLUE, DrInsight, Alakwaa's, Guo's) per dataset, cluster, or simulation setting
- name: ASGARD
  role: Pipeline under benchmark whose GetDrug/DrugScore-ranked drug lists are evaluated
    as ROC/AUC against ground truth
  repo: https://github.com/lanagarmire/ASGARD
- name: DrInsight
  role: Bulk-cell baseline (version 0.1.1, CEG-based ranking) whose predictions are
    scored as ROC/AUC comparison points
provenance:
  source_task_ids:
  - task_004
  - task_001
  - task_002
  - task_003
  source_papers:
  - doi: 10.1038/s41467-023-36637-3
    title: ASGARD is A Single-cell Guided Pipeline to Aid Repurposing of Drugs
  source_content_hash: sha256:e48b429d043b58d844a23453a953f334d0155edbf3465a459b54ed3b3db949f8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lanagarmire/ASGARD
  tools:
  - ASGARD
  - CLUE
  - DrInsight
  - SingleR
  - pROC
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# roc-auc-curve-evaluation

## Summary

Benchmarking of drug repurposing pipelines by computing ROC curves and area under the curve (AUC) with the pROC package, using drugs in advanced clinical trials (Supplementary Data 4) as positive cases to quantify how well ASGARD or baseline methods rank known ground-truth drugs.

## When to use

Use when comparing ranked drug/compound predictions from two or more pipelines (e.g., ASGARD multi-cluster score vs. CLUE, DrInsight, Alakwaa's, Guo's) against a known positive-drug ground truth and the comparison must be quantified as ROC/AUC with statistical significance testing.

## When NOT to use

- When no ground-truth positives exist — e.g., exploratory drug recommendations without a Supplementary-Data-4-style advanced-trial drug list to define positive cases
- When comparing methods by ranking metrics other than binary classification performance (e.g., drug-response correlation validated via TRANSACT with TNBC patient samples), where ROC/AUC against trial-status positives is not the appropriate yardstick

## Inputs

- Per-pipeline ranked drug/compound prediction lists (ASGARD DrugScore/TopDrug output, CLUE rankings, DrInsight v0.1.1 rankings, Alakwaa's and Guo's cluster-level rankings)
- Positive-case ground-truth drug list (Supplementary Data 4: drugs in advanced clinical trials or animal models)
- Per-cluster or per-simulation replicate AUC values for group-level comparisons

## Outputs

- ROC curves and AUC values per pipeline, dataset, cluster, or DE method
- Averaged AUC summaries per comparison group (e.g., ASGARD 0.83 vs. Alakwaa's 0.76 vs. Guo's 0.54 across breast cancer clusters)
- Student's t-test P-values for AUC differences (significant at P < 0.05)
- Plotted ROC comparison figures (e.g., supplementary_fig3_robust_auc_curves.pdf)

## How to apply

Assemble per-pipeline ranked drug lists (e.g., ASGARD GetDrug/DrugScore output, CLUE or DrInsight rankings) and designate positives from Supplementary Data 4 — drugs in advanced clinical trials or with animal-model evidence. Compute ROC curves and AUCs for each pipeline using the pROC package, per dataset, per cluster, or per simulation parameter-grid setting as appropriate. For the DE-method comparison, average AUCs across the breast cancer, leukemia, and COVID-19 datasets; for the single-cluster comparison, average per-cluster AUCs across the 8 breast, 4 Pre-T ALL, and 4 COVID-19 cell types. Assess significance of AUC differences with a Student's t-test (P < 0.05), and validate that reproduced values match reported figures (e.g., ASGARD multi-cluster AUCs of 0.92/0.95/0.88 for breast cancer, leukemia, and COVID-19; Limma-Bayes DE yielding ~0.90–0.92).

## Related tools

- **pROC** (R package used to compute ROC curves and AUCs for every pipeline (ASGARD, CLUE, DrInsight, Alakwaa's, Guo's) per dataset, cluster, or simulation setting)
- **ASGARD** (Pipeline under benchmark whose GetDrug/DrugScore-ranked drug lists are evaluated as ROC/AUC against ground truth) — https://github.com/lanagarmire/ASGARD
- **DrInsight** (Bulk-cell baseline (version 0.1.1, CEG-based ranking) whose predictions are scored as ROC/AUC comparison points)

## Examples

```
library(pROC); roc_obj <- roc(labels, asgard_scores, levels=c(0,1), direction="<"); auc(roc_obj); t.test(aucs_asgard, aucs_baseline)
```

## Evaluation signals

- Reproduced multi-cluster ASGARD AUCs should match reported values: 0.92 (breast), 0.95 (leukemia), 0.88 (COVID-19), vs. CLUE 0.74/0.82/0.85 and DrInsight 0.81/0.73/0.73
- Single-cluster averages across 8 breast cancer clusters should reproduce ASGARD 0.83 (0.80–0.86), Alakwaa's 0.76 (0.68–0.83), Guo's 0.54 (0.47–0.56)
- Multi-cluster ASGARD drug score must yield significantly better AUC than individual-cluster predictions (Student's t-test, P < 0.05)
- DE-method AUC ordering must match the benchmark: Limma (Bayes) best (~0.90–0.92), edgeR (~0.83–0.86) and Seurat Wilcoxon (~0.80–0.86) intermediate, DESeq2 lowest
- Simulation robustness curves must show stable AUCs across cell-population sizes, DE levels (20%–90%), and diseased-cell proportions (20%–90%)

## Limitations

- Positive-case ground truth relies on Supplementary Data 4 (drugs in advanced trials/animal models), which is not enumerated in the text, so AUC estimates depend on the completeness of that external list
- Quantitative AUC results are primarily referenced via figures (Figs. 2, 3, Supplementary Fig. 2) rather than numerically reported in the text
- AUC performance varies with the differential expression method chosen and with cell population sizes, proportions, and DE levels, requiring careful per-dataset selection and robustness simulations

## Evidence

- [discussion] ROCs and AUCs are calculated for each pipeline using the pROC package: "ROCs and AUCs are calculated for each pipeline using the pROC package79"
- [discussion] Significant AUC of multi-cluster score: "The prediction using the multi-cluster drug score shows a significantly (P-value < 0.05, Student’s t test) better AUC than the prediction based on individual clusters"
- [other] pROC used across simulation parameter grid: [evidence span withheld — anchor status: not_found]
- [discussion] Positive-case ground truth source: "all the drugs and compounds we found are listed in Supplementary Data 4"
- [discussion] AUC results referenced via figures: "ASGARD performs much better than all these methods in predicting drugs/compounds (Figs. 2, 3, Supplementary Fig. 2)"
