---
name: drug-wise-spearman-correlation-filtering
description: Use when evaluating CaDRReS-Sc drug response predictions trained on GDSC/PRISM
  cell line data and you need to remove poorly predicted drugs (bottom-quartile drug-wise
  Spearman correlation) before reporting results.
when_to_use_negative:
- When selecting feature genes for the PRISM kernel — that uses a different filter
  (8,087 genes with |Pearson correlation| ≥ 0.2 between expression and PRISM AUC),
  not a drug-wise Spearman quartile filter
- When the drug set is very small (e.g., a single-drug validation like the idasanutlin
  24-cell-line experiment), since dropping the bottom quartile of drugs would remove
  the drug of interest entirely
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: scDrug
  role: Pipeline whose drug response prediction module trains the CaDRReS-Sc models
    and applies the drug-wise Spearman quartile filter before reporting IC50/AUC predictions
  repo: https://github.com/ailabstw/scDrug
- name: CaDRReS-Sc
  role: Recommender-system framework providing the GDSC response data and the model
    whose per-drug prediction quality is assessed by the Spearman correlation filter
  repo: https://github.com/CSB5/CaDRReS-SC
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.csbj.2022.11.055
    title: 'scDrug: From single-cell RNA-seq to drug response prediction'
  source_content_hash: sha256:6f9558287dcf8563a310dce7be8b88e97fa470dd96c46356d2c090efbb710045
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ailabstw/scDrug
  tools:
  - CaDRReS-Sc
  - Python
  - scDrug
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# Drug-wise Spearman correlation filtering

## Summary

A post-training quality filter used in the scDrug drug response prediction pipeline: after training CaDRReS-Sc models on GDSC (226 drugs, 1074 cell lines) and PRISM (1448 drugs, 480 cell lines) data, drug-wise Spearman correlations between actual and predicted response are computed, and drugs whose coefficient falls below the first-quartile coefficient are dropped before downstream analysis.

## When to use

Use when evaluating CaDRReS-Sc drug response predictions trained on GDSC/PRISM cell line data and you need to remove poorly predicted drugs (bottom-quartile drug-wise Spearman correlation) before reporting results.

## When NOT to use

- When selecting feature genes for the PRISM kernel — that uses a different filter (8,087 genes with |Pearson correlation| ≥ 0.2 between expression and PRISM AUC), not a drug-wise Spearman quartile filter
- When the drug set is very small (e.g., a single-drug validation like the idasanutlin 24-cell-line experiment), since dropping the bottom quartile of drugs would remove the drug of interest entirely

## Inputs

- Trained CaDRReS-Sc model predictions (predicted response per drug per cell line; 10-dim latent space for GDSC, 140-dim for PRISM)
- GDSC response data (measured IC50) of 226 drugs in 1074 cancer cell lines provided by CaDRReS-Sc
- PRISM Repurposing dataset version 19Q4 responses of 1448 drugs against 480 cell lines, response scale D′ = 2.40D − 1.20 (1 − AUC)

## Outputs

- Filtered drug set (drugs with drug-wise Spearman coefficient at or above the first-quartile coefficient)
- Cleaned prediction tables used for downstream outputs such as IC50_prediction.csv and AUC_prediction.csv

## How to apply

After training the modified CaDRReS-Sc models (drug bias only, L2 regularization; 10-dimensional latent space for GDSC on −log(IC50) from the CaDRReS-Sc-provided 226 drugs × 1074 cell lines GDSC response data, and 140-dimensional latent space for PRISM on scaled 1 − AUC from the 19Q4 Repurposing dataset, learning rate 0.01, up to 100,000 epochs), compute a Spearman correlation coefficient per drug between its actual and predicted response values across cell lines. Then identify the first quartile of these per-drug coefficients and drop all drugs whose coefficient is lower than that first-quartile coefficient. The rationale is that drugs whose response structure the recommender model cannot capture would contaminate downstream predictions (e.g., cluster-wise IC50/AUC rankings), so the filter retains only the ~75% best-predicted drugs for prediction outputs like IC50_prediction.csv or AUC_prediction.csv.

## Related tools

- **scDrug** (Pipeline whose drug response prediction module trains the CaDRReS-Sc models and applies the drug-wise Spearman quartile filter before reporting IC50/AUC predictions) — https://github.com/ailabstw/scDrug
- **CaDRReS-Sc** (Recommender-system framework providing the GDSC response data and the model whose per-drug prediction quality is assessed by the Spearman correlation filter) — https://github.com/CSB5/CaDRReS-SC

## Examples

```
python3 drug_response_prediction.py -i scanpyobj.h5ad -o ./ -m GDSC
```

## Evaluation signals

- The retained drug set should contain exactly the drugs whose actual-vs-predicted drug-wise Spearman coefficient is at or above the first quartile of all per-drug coefficients (≈75% of drugs)
- The filter must be computed per drug (each drug correlated across cell lines), not per cell line
- After filtering, the overall Pearson correlation between PRISM (1 − AUC) and GDSC (−log(IC50)) values across shared drugs should reproduce the reported value of 0.615 (prism_gdsc_correlation.csv)
- Response scales must be transformed before correlation: GDSC IC50 → −log(IC50), PRISM AUC → scaled 1 − AUC (D′ = 2.40D − 1.20)
- Downstream prediction files (IC50_prediction.csv, AUC_prediction.csv) should contain only drugs that survived the quartile filter

## Limitations

- Drug response prediction in scDrug is limited to experimentally screened compounds, so the filter only operates over drugs present in GDSC/PRISM screening data
- The first-quartile cutoff is a fixed relative threshold; it discards 25% of drugs regardless of absolute prediction quality

## Evidence

- [methods] drug-wise Spearman correlation filtering: "we dropped the drugs whose drug-wise coefficient was lower than the first quartile coefficient."
- [methods] GDSC response data input: "we used the response data (measured IC50) of 226 drugs in 1074 cancer cell lines, provided by CaDRReS-Sc from the GDSC database"
- [methods] PRISM response scale: "we used the PRISM Repurposing dataset (version 19Q4) as the training data, which contained the responses of 1448 drugs against 480 cell lines"
- [readme] CaDRReS-Sc prediction context: "reported clusterwise IC50 and cell death percentages to drugs in the GDSC database via [CaDRReS-Sc]"
