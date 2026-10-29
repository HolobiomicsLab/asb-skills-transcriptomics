---
name: cross-dataset-drug-response-scale-transformation
description: Use when drug response data come from datasets with incompatible units
  (e.g., GDSC IC50 vs PRISM Repurposing 19Q4 AUC) and must be placed on a common sensitivity
  scale before model training or cross-dataset correlation.
when_to_use_negative:
- Both datasets already share one response scale (e.g., all IC50, or all AUC with
  identical normalization) — re-applying −log(IC50) or D′ = 2.40D − 1.20 would distort
  the values.
- Predicting response for a compound absent from the GDSC/PRISM screens — scDrug's
  prediction is limited to experimentally screened compounds, so no transformed scale
  exists for it.
- Single-dataset IC50 prediction with the pretrained GDSC model only (drug_response_prediction.py
  --model GDSC), where no cross-dataset harmonization is required.
edam_operation: http://edamontology.org/operation_3465
edam_topics:
- http://edamontology.org/topic_0154
tools:
- name: scDrug
  role: Workflow whose drug response prediction module consumes GDSC IC50 and PRISM
    AUC data after scale transformation to train/apply response models
  repo: https://github.com/ailabstw/scDrug
- name: CaDRReS-Sc
  role: Supplies the GDSC IC50 training response data (226 drugs × 1074 cell lines)
    and the recommender-system model whose training and evaluation rely on the transformed
    scales
  repo: https://github.com/CSB5/CaDRReS-SC
- name: Python
  role: Environment in which the −log(IC50) and D′ = 2.40D − 1.20 transformations
    and the Pearson correlation computation are implemented
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.csbj.2022.11.055
    title: 'scDrug: From single-cell RNA-seq to drug response prediction'
  source_content_hash: sha256:8e9d4ee25a8f78bd2bc76e67fe27fac74e3fa460e3a498e97d7a431322477c03
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ailabstw/scDrug
  tools:
  - CaDRReS-Sc
  - Python
  - scDrug
  edam_operation: http://edamontology.org/operation_3465
  edam_topics:
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# cross-dataset-drug-response-scale-transformation

## Summary

Harmonizes pharmacological response measurements recorded in different units — GDSC IC50 and PRISM Repurposing 19Q4 AUC — onto a shared sensitivity scale via −log(IC50) and the linear rescaling D′ = 2.40D − 1.20 of 1 − AUC. This makes GDSC-trained and PRISM-trained response models comparable, with an expected cross-dataset Pearson correlation of 0.615.

## When to use

Use when drug response data come from datasets with incompatible units (e.g., GDSC IC50 vs PRISM Repurposing 19Q4 AUC) and must be placed on a common sensitivity scale before model training or cross-dataset correlation.

## When NOT to use

- Both datasets already share one response scale (e.g., all IC50, or all AUC with identical normalization) — re-applying −log(IC50) or D′ = 2.40D − 1.20 would distort the values.
- Predicting response for a compound absent from the GDSC/PRISM screens — scDrug's prediction is limited to experimentally screened compounds, so no transformed scale exists for it.
- Single-dataset IC50 prediction with the pretrained GDSC model only (drug_response_prediction.py --model GDSC), where no cross-dataset harmonization is required.

## Inputs

- GDSC drug response matrix of measured IC50 values for 226 drugs across 1074 cancer cell lines, as provided by CaDRReS-Sc
- PRISM Repurposing dataset version 19Q4 AUC response matrix for 1448 drugs against 480 cell lines (DepMap Portal)

## Outputs

- GDSC sensitivity matrix in −log(IC50) scale
- PRISM sensitivity matrix in scaled 1 − AUC scale (D′ = 2.40D − 1.20)
- prism_gdsc_correlation.csv containing the overall Pearson correlation between the two transformed response scales across shared drugs

## How to apply

Load the raw response matrices: GDSC measured IC50 for 226 drugs in 1074 cancer cell lines (provided by CaDRReS-Sc) and PRISM Repurposing 19Q4 AUC for 1448 drugs against 480 cell lines. Invert both scales toward sensitivity: convert GDSC IC50 to −log(IC50) so lower IC50 (higher potency) yields larger scores, and convert PRISM AUC to 1 − AUC, then rescale with the linear mapping D′ = 2.40D − 1.20. Keep the transformations row/column-preserving so drug and cell line identities survive for downstream kernel feature construction from CCLE 21Q3 expression (17,419 common genes for GDSC, 8,087 genes with ≥0.2 absolute expression–AUC Pearson correlation for PRISM) and CaDRReS-Sc model training (10-dimensional GDSC and 140-dimensional PRISM latent spaces, learning rate 0.01, up to 100,000 epochs). Validate the harmonization by computing the overall Pearson correlation between PRISM (1 − AUC) and GDSC (−log(IC50)) across shared drugs — it should reproduce the reported 0.615 — and write the result to prism_gdsc_correlation.csv.

## Related tools

- **scDrug** (Workflow whose drug response prediction module consumes GDSC IC50 and PRISM AUC data after scale transformation to train/apply response models) — https://github.com/ailabstw/scDrug
- **CaDRReS-Sc** (Supplies the GDSC IC50 training response data (226 drugs × 1074 cell lines) and the recommender-system model whose training and evaluation rely on the transformed scales) — https://github.com/CSB5/CaDRReS-SC
- **Python** (Environment in which the −log(IC50) and D′ = 2.40D − 1.20 transformations and the Pearson correlation computation are implemented)

## Examples

```
gdsc_score = -np.log(ic50); prism_score = 2.40 * (1 - prism_auc) - 1.20
```

## Evaluation signals

- GDSC ordering check: −log(IC50) must invert the IC50 ranking — a cell line with lower IC50 must receive a strictly larger sensitivity score than a more resistant one.
- PRISM mapping check: D′ = 2.40D − 1.20 applied to D = 1 − AUC must send D = 0.5 to D′ = 0.0 and preserve the ordering of 1 − AUC values.
- The overall Pearson correlation between PRISM (1 − AUC) and GDSC (−log(IC50)) across shared drugs must reproduce the reported value of 0.615 in prism_gdsc_correlation.csv.
- Dimensions preserved: 226 GDSC drugs × 1074 cell lines and 1448 PRISM drugs × 480 cell lines before and after transformation; only values change, not drug/cell line labels.
- Downstream drug-wise Spearman filter must drop drugs whose coefficient falls below the first-quartile coefficient from the transformed matrices before model evaluation.

## Limitations

- GDSC (226 drugs, 1074 cell lines) and PRISM 19Q4 (1448 drugs, 480 cell lines) cover different drug and cell line panels, so cross-dataset comparison is restricted to the shared drug subset.
- D′ = 2.40D − 1.20 is an empirical linear rescaling of PRISM 1 − AUC, not a conversion to IC50; the scales are comparable only through correlation, not absolute equivalence.
- scDrug's drug response prediction is limited to experimentally screened compounds, so the transformed scales cannot be extended to unscreened drugs.

## Evidence

- [other] Scale transformation step defining −log(IC50) and D′ = 2.40D − 1.20: [evidence span withheld — anchor status: not_found]
- [methods] Provenance and dimensions of the GDSC response data: "we used the response data (measured IC50) of 226 drugs in 1074 cancer cell lines, provided by CaDRReS-Sc from the GDSC database"
- [methods] Provenance and dimensions of the PRISM response data: "we used the PRISM Repurposing dataset (version 19Q4) as the training data, which contained the responses of 1448 drugs against 480 cell lines"
- [other] Cross-dataset validation output and expected correlation target: [evidence span withheld — anchor status: not_found]
- [readme] README confirmation that GDSC IC50 and PRISM AUC are the response units in the prediction module: "reported clusterwise IC50 and cell death percentages to drugs in the GDSC database"
