---
name: fdr-multiple-testing-first-step-screening
description: Use when regression covariates number in the thousands to ~hundred thousand
  (TCGA mRNA entries or mutated genes), the multiple-testing problem is of concern,
  and variables must be screened before penalized regression.
when_to_use_negative:
- The covariate set is small enough that the multiple-testing problem is negligible
  — the screening step is optional and only recommended when variables reach the magnitude
  of a hundred thousand
- The data have not yet passed the preliminary frequency screening (genes mutated
  in ≥1% or mRNA entries abnormal in ≥2% of patients with Z > 2 or Z < −2)
- The analysis goal is coefficient-level inference requiring post-selection p-values
  rather than variable pre-filtering (use selectiveInference for that instead)
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0622
tools:
- name: HDMAC
  role: Web-based/Shiny platform and R scripts offering FDR control as an optional
    first-step screening method before penalized regression
  repo: https://github.com/chung-R/HD-MAC
- name: R
  role: Language in which the FDR screening, univariate regression fits, and penalized
    model refitting are implemented
- name: Shiny
  role: R package powering the interactive HDMAC web interface for choosing first-step
    screening options
provenance:
  source_task_ids:
  - task_004
  - task_006
  - task_002
  - task_001
  source_papers:
  - doi: 10.1038/s41598-020-60791-z
    title: 'HDMAC: A Web-Based Interactive Program for High-Dimensional Analysis of
      Molecular Alterations in Cancer'
  source_content_hash: sha256:ad3e3e8358e7d0a29b21e868dba352a55dbd987562d9beedfa2b07817b968544
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/chung-R/HD-MAC
  tools:
  - HDMAC
  - R
  - Shiny
  - selectiveInference
  edam_operation: http://edamontology.org/operation_3660
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# fdr-multiple-testing-first-step-screening

## Summary

An optional first-step Benjamini-Hochberg FDR screening in HDMAC that filters unrelated high-dimensional genetic variables (univariate regression p-value per gene) before fitting penalized Ridge/Lasso/adaptive Lasso Cox or logistic regression models.

## When to use

Use when regression covariates number in the thousands to ~hundred thousand (TCGA mRNA entries or mutated genes), the multiple-testing problem is of concern, and variables must be screened before penalized regression.

## When NOT to use

- The covariate set is small enough that the multiple-testing problem is negligible — the screening step is optional and only recommended when variables reach the magnitude of a hundred thousand
- The data have not yet passed the preliminary frequency screening (genes mutated in ≥1% or mRNA entries abnormal in ≥2% of patients with Z > 2 or Z < −2)
- The analysis goal is coefficient-level inference requiring post-selection p-values rather than variable pre-filtering (use selectiveInference for that instead)

## Inputs

- High-dimensional covariate matrix of TCGA genomic entries (e.g., 9,548 Z-scored mRNA expression entries or 670 mutated genes retained after ≥1%/≥2% frequency pre-screening)
- Outcome vector: overall survival time with censoring indicators (Cox models) or binary labels such as invasion subtype (logistic models)

## Outputs

- List of FDR-passing candidate entries/genes (e.g., ZSWIM8 and PABPC3 for ovarian mutation data)
- Screened covariate matrix used to refit penalized regression models
- Comparison metrics (C-index or AUC) for models with vs. without FDR screening, e.g., cindex_table.csv or auc_table.csv

## How to apply

For each genetic variable, fit a univariate regression appropriate to the outcome — univariate Cox regression per gene for overall survival with censoring indicators, or univariate logistic regression per gene for binary outcomes like bladder cancer invasion subtype — and compute its p-value. Apply Benjamini-Hochberg FDR control at the default threshold of 0.05 and retain only candidate entries passing the threshold (e.g., the 2 mutated genes ZSWIM8 and PABPC3 in the TCGA ovarian HGSC mutation analysis, or the 24 mRNA entries in the bladder cancer adaptive Lasso analysis). Refit the penalized models (ridge, Lasso, adaptive Lasso) on the screened entries only, then estimate prediction power with 5-fold cross validation (C-index for survival, AUC for binary outcomes). Rationale: penalized regression models fall short when variables reach the magnitude of a hundred thousand, and FDR screening addresses the multiple-comparison issue arising with large-volume genomic data.

## Related tools

- **HDMAC** (Web-based/Shiny platform and R scripts offering FDR control as an optional first-step screening method before penalized regression) — https://github.com/chung-R/HD-MAC
- **R** (Language in which the FDR screening, univariate regression fits, and penalized model refitting are implemented)
- **Shiny** (R package powering the interactive HDMAC web interface for choosing first-step screening options)

## Examples

```
Rscript -e "p <- apply(X, 2, function(x) summary(coxph(Surv(time, status) ~ x))$coef[,'Pr(>|z|)']); keep <- names(p.adjust(p, method='BH')[p.adjust(p, method='BH') < 0.05]); Xs <- X[, keep]"
```

## Evaluation signals

- The FDR-screened ovarian mutation analysis must select exactly 2 genes: ZSWIM8 (HR 7.493) and PABPC3 (HR 5.635)
- The bladder cancer adaptive Lasso pipeline on FDR-screened entries must yield exactly 24 selected mRNA entries reproducing Table 4 (e.g., PSORS1C2 coefficient 1.14, ICOSLG p = 0.004)
- With- and without-screening C-index tables must reproduce reported values (ovarian mRNA: 0.591/0.554/0.560 without FDR vs 0.538/0.538/0.540 with FDR; ovarian mutation: 0.529/0.501/0.501 vs 0.502/0.502/0.497)
- Screened bladder cancer AUC values must match reported percentages (ridge 66.515, Lasso 67.020, adaptive Lasso 64.300 with FDR)
- The BH procedure must use the default FDR threshold of 0.05 and univariate regression p-values per gene appropriate to the outcome type

## Limitations

- FDR screening is recommended when variables reach the magnitude of a hundred thousand, but screening changes which genes enter the final model and can lower prediction power (e.g., ovarian mRNA ridge C-index drops from 0.591 to 0.538)
- Screening is applied after the preliminary frequency filter; genes whose mutations are found in <1% or mRNA entries abnormal in <2% of patients are removed regardless of FDR status
- Genes identified are preliminary and require further in-depth study; genome-wide search for genetic changes associated with ovarian cancer survival is still waiting

## Evidence

- [abstract] Choice of a first-step screening is provided to address the multiple-comparison issue that often arises with large-volume genomic data.: "Choice of a first-step screening is provided to address the multiple-comparison issue that often arises with large-volume genomic data."
- [methods] FDR as optional first-step screening: "the method to control the FDR is provided as an optional first-step screening method"
- [discussion] Penalized regression falls short at huge variable counts: "Inclusion of FDR is recommended if users are dealing with variables at the magnitude of a hundred thousand where penalized regression models fall short"
- [intro] Screening choice objective: [evidence span withheld — anchor status: not_found]
- [methods] Preliminary frequency screening thresholds: "a preliminary screening was performed so that only the genes whose mutations were found in 1% or more and the mRNA entries whose abnormal expression was found in 2% or more of the patients were"
