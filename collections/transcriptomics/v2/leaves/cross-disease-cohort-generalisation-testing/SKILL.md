---
name: cross-disease-cohort-generalisation-testing
description: Use when a severity biomarker score built from one disease's transcriptomic
  cohorts must be validated for discriminative power (severe vs mild) in independent
  cohorts of other pathogens, treating AUC~0.5 as no discrimination.
when_to_use_negative:
- The goal is identifying differentially regulated genes within COVID-19 cohorts rather
  than testing a fixed two-gene score in external-pathogen cohorts
- Cohorts lack severe/mild phenotype labels, so a positive class cannot be defined
  for ROC analysis
- The biomarkers of interest are proteins or clinical measurements rather than transcriptomic
  MCEMP1/HLA-DRA expression
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3517
- http://edamontology.org/topic_3308
tools:
- name: scikit-learn
  role: roc_curve/roc_auc_score used to compute ROC curves and AUC of the combined
    Z-score per cohort
- name: Streamlit
  role: Webtool platform used to publish the underlying COVID-19 transcriptomics database
    queried for the gene-expression data
  repo: https://github.com/kuanrongchan/COVID19-severity
- name: pip
  role: Installs dependencies (requirements.txt) to run the app/repository code locally
  repo: https://github.com/kuanrongchan/COVID19-severity
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.ebiom.2023.104472
    title: Early peripheral blood MCEMP1 and HLA-DRA expression predicts COVID-19
      prognosis
  source_content_hash: sha256:1a21061d5166975aa4010f552e94833543ee5e2604f717565d616731e7a38877
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kuanrongchan/COVID19-severity
  tools:
  - Streamlit
  - pip
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3517
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# cross-disease-cohort-generalisation-testing

## Summary

Tests whether a gene-expression severity score derived in COVID-19 cohorts (combined MCEMP1/HLA-DRA Z-score) generalises to independent cohorts of other viral infections (influenza, RSV, dengue) by recomputing ROC/AUC per cohort.

## When to use

Use when a severity biomarker score built from one disease's transcriptomic cohorts must be validated for discriminative power (severe vs mild) in independent cohorts of other pathogens, treating AUC~0.5 as no discrimination.

## When NOT to use

- The goal is identifying differentially regulated genes within COVID-19 cohorts rather than testing a fixed two-gene score in external-pathogen cohorts
- Cohorts lack severe/mild phenotype labels, so a positive class cannot be defined for ROC analysis
- The biomarkers of interest are proteins or clinical measurements rather than transcriptomic MCEMP1/HLA-DRA expression

## Inputs

- Accession numbers of influenza, RSV and dengue cohorts from Supplementary Table S5
- Severe-vs-mild gene expression matrices downloaded from GEO/Mendeley Data (9 cohorts across 3 diseases)
- Gene names of interest: MCEMP1 and HLA-DRA expression values per subject

## Outputs

- Per-cohort ROC curves and AUC values for the combined MCEMP1/HLA-DRA Z-score
- Summary table roc_results.csv with AUC, confidence direction and discrimination outcome per cohort and disease

## How to apply

Retrieve accession numbers for influenza, RSV and dengue transcriptomic cohorts from Supplementary Table S5 and download severe-vs-mild expression matrices from GEO or Mendeley Data. Per cohort, z-score normalise MCEMP1 and HLA-DRA expression across samples and average the two gene Z-scores per subject to form the combined Z-score severity metric. Using scikit-learn's roc_curve/roc_auc_score, compute ROC curves and AUC values treating severe status as the positive class across each of the 9 cohorts. Tabulate AUC, confidence direction and discrimination outcome (significant vs no discrimination, AUC~0.5) per cohort and disease, and save as roc_results.csv. Validation requires the reproduced pattern to match the paper: severe influenza discriminated in only 2 of 3 cohorts, severe RSV discriminated in 2 of 2 studies, and no discrimination for dengue.

## Related tools

- **scikit-learn** (roc_curve/roc_auc_score used to compute ROC curves and AUC of the combined Z-score per cohort)
- **Streamlit** (Webtool platform used to publish the underlying COVID-19 transcriptomics database queried for the gene-expression data) — https://github.com/kuanrongchan/COVID19-severity
- **pip** (Installs dependencies (requirements.txt) to run the app/repository code locally) — https://github.com/kuanrongchan/COVID19-severity

## Examples

```
from sklearn.metrics import roc_curve, roc_auc_score; fpr, tpr, _ = roc_curve(y_severe, combined_zscore); auc = roc_auc_score(y_severe, combined_zscore)
```

## Evaluation signals

- Reproduced pattern matches the paper: severe influenza discriminated in only 2 of 3 cohorts
- Severe RSV discriminated in 2 independent studies (2 of 2)
- No discrimination observed for dengue (AUC~0.5 in all dengue cohorts)
- roc_results.csv contains per-cohort AUC, confidence direction and discrimination outcome columns for all 9 cohorts
- Per-subject combined score equals the mean of the per-gene Z-scores of MCEMP1 and HLA-DRA

## Limitations

- The score discriminates severe disease inconsistently across pathogens (partial influenza success, no dengue signal), indicating pathogen-specific generalisation limits
- Sensitivity and specificity of MCEMP1/HLA-DRA expression have not been validated prospectively; future prospective studies are needed
- Heterogeneity in the study datasets was noted, which may influence MCEMP1 and HLA-DRA expression across cohorts

## Evidence

- [results] Applying the combined MCEMP1/HLA-DRA score, severe from mild influenza was discriminated in only 2 of 3 cohorts, severe RSV was discriminated in 2 independent studies, and no discrimination was observed for dengue.: [evidence span withheld — anchor status: not_found]
- [methods] Per cohort, z-score normalise MCEMP1 and HLA-DRA expression across samples and average the two gene Z-scores per subject: [evidence span withheld — anchor status: not_found]
- [methods] Compute ROC curves and AUC values for the combined Z-score in each of the 9 cohorts using scikit-learn's roc_curve/roc_auc_score: [evidence span withheld — anchor status: not_found]
- [discussion] Future prospective studies will also be needed to validate the sensitivity and specificity of using MCEMP1 and HLA-DRA expression levels in predicting severe COVID-19.: "Future prospective studies will also be needed to validate the sensitivity and specificity of using MCEMP1 and HLA-DRA expression levels in predicting severe COVID-19."
- [readme] This database is a curation of 7 transcriptomics datasets comparing the gene expression in severe COVID-19 and mild COVID-19 subjects.: "This database is a curation of 7 transcriptomics datasets comparing the gene expression in severe COVID-19 and mild COVID-19 subjects."
