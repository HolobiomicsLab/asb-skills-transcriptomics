---
name: spearman-correlation-analysis
description: Use when validating predicted per-cluster or per-lineage drug sensitivity
  scores (e.g. AUC_prediction.csv from scDrug --model PRISM) against an observed ordered
  response such as treated-vs-control cell-proportion changes, without assuming linearity.
when_to_use_negative:
- No paired predictions and observations exist yet, e.g. AUC_prediction.csv has not
  been generated or observed post-treatment proportions are unavailable
- The comparison variables are binary/categorical rather than ordered, where rank
  correlation is not the appropriate validation statistic
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: scDrug
  role: Produces the AUC_prediction.csv drug sensitivity scores (PRISM model) that
    are correlated against observed proportion changes
  repo: https://github.com/ailabstw/scDrug
- name: CaDRReS-Sc
  role: Underlying recommender-system framework whose log2 fold-change feature requirements
    feed the sensitivity predictions being validated
  repo: https://github.com/CSB5/CaDRReS-SC
- name: Scanpy
  role: Provides the scanpyobj.h5ad AnnData object from which cluster proportions
    and expression features are derived
- name: Python
  role: Environment in which scipy.stats.spearmanr is run to compute the rank correlation
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.csbj.2022.11.055
    title: 'scDrug: From single-cell RNA-seq to drug response prediction'
  source_content_hash: sha256:9fff869646bba9eb6922ae1cb8f9c9e5240c6e5c69bfd7e7b93cfb7acc3a85e7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ailabstw/scDrug
  tools:
  - CaDRReS-Sc
  - Python
  - Scanpy
  - scDrug
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# spearman-correlation-analysis

## Summary

Non-parametric rank-correlation analysis using scipy.stats.spearmanr to validate that scDrug/CaDRReS-Sc-predicted drug sensitivity scores agree with observed biological responses, e.g. correlating predicted idasanutlin AUC per cell lineage with observed tumor-cluster proportion changes between treated and control samples.

## When to use

Use when validating predicted per-cluster or per-lineage drug sensitivity scores (e.g. AUC_prediction.csv from scDrug --model PRISM) against an observed ordered response such as treated-vs-control cell-proportion changes, without assuming linearity.

## When NOT to use

- No paired predictions and observations exist yet, e.g. AUC_prediction.csv has not been generated or observed post-treatment proportions are unavailable
- The comparison variables are binary/categorical rather than ordered, where rank correlation is not the appropriate validation statistic

## Inputs

- AUC_prediction.csv with predicted sensitivity (scaled 1-AUC) per cluster from drug_response_prediction.py --model PRISM
- Per-lineage observed tumor-cluster proportions from treated and control samples of the 24 idasanutlin-treated cell lines (scanpyobj.h5ad-derived)
- Aggregated paired vectors: per-lineage predicted sensitivity scores and per-lineage observed proportion changes

## Outputs

- Spearman rank correlation coefficient (expected approx. -0.42 for the idasanutlin case)
- Two-sided p-value (expected approx. 0.04)
- Validation report stating whether predicted sensitivity is significantly negatively associated with observed proportion changes

## How to apply

Aggregate the per-cluster predicted sensitivity scores from AUC_prediction.csv (produced by drug_response_prediction.py with the pre-trained PRISM model, 140-dimensional latent space, learned variable scaled 1-AUC via D' = 240D - 120) by cell lineage, and compute observed proportion changes of tumor clusters between treated and control samples. Compute the Spearman rank correlation between the two per-lineage vectors with scipy.stats.spearmanr. Rank-based correlation is chosen because predicted sensitivity and proportion changes need not be linearly related, only monotonically. A significantly negative coefficient is the expected validation signal: lineages predicted more sensitive to idasanutlin should show larger drops in tumor-cluster proportion. In the scDrug idasanutlin case study this reproduced Spearman's correlation coefficient = -0.42 with p-value = 0.04.

## Related tools

- **scDrug** (Produces the AUC_prediction.csv drug sensitivity scores (PRISM model) that are correlated against observed proportion changes) — https://github.com/ailabstw/scDrug
- **CaDRReS-Sc** (Underlying recommender-system framework whose log2 fold-change feature requirements feed the sensitivity predictions being validated) — https://github.com/CSB5/CaDRReS-SC
- **Scanpy** (Provides the scanpyobj.h5ad AnnData object from which cluster proportions and expression features are derived)
- **Python** (Environment in which scipy.stats.spearmanr is run to compute the rank correlation)

## Examples

```
from scipy.stats import spearmanr; rho, p = spearmanr(pred_sensitivity_by_lineage, observed_proportion_changes)
```

## Evaluation signals

- Computed Spearman correlation coefficient reproduces the reported -0.42 for predicted idasanutlin sensitivity vs. observed proportion changes
- Associated p-value is approximately 0.04, i.e. the negative association is significant at p < 0.05
- AUC_prediction.csv exists and contains one predicted sensitivity value per cluster (output of --model PRISM)
- Both correlated vectors are aggregated consistently per cell lineage, with matched lineage labels across prediction and observation

## Limitations

- Drug response prediction in scDrug is limited to experimentally screened compounds, so this correlation-based validation cannot extend to untested drugs
- Available scRNA-seq data with drug treatments are short for evaluation; the correlation is based on only 24 cell lines, limiting statistical power
- De novo prediction of drug sensitivity is not yet implemented and extensive experimental validations of the pipeline are still necessary

## Evidence

- [results] Negative correlation validating PRISM sensitivity predictions: "the proportion changes were negatively correlated with the predicted sensitivity scores significantly (Spearman's correlation coefficient = −0.42 with p-value = 0.04)"
- [readme] PRISM prediction output file: "`AUC_prediction.csv` whlie using parameter `--model PRISM`"
- [intro] Prediction framework context: "recent studies have leveraged scRNA-seq and bulk transcriptomic profiles to predict drug response for heterogeneous tumor cells, such as Beyondcell and CaDRReS-Sc"
