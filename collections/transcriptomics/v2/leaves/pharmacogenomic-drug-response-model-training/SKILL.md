---
name: pharmacogenomic-drug-response-model-training
description: Use when bulk drug response models must be trained or applied on GDSC/PRISM
  screening data with CCLE expression kernel features, e.g. reproducing cross-database
  correlation or scoring cluster-wise IC50/AUC predictions.
when_to_use_negative:
- Drug response prediction is limited to experimentally screened compounds — de novo
  sensitivity prediction for unscreened drugs is not supported
- Input is single-cell raw data needing preprocessing/clustering first; this skill
  starts from response tables plus CCLE expression or a completed scanpyobj.h5ad
- Available scRNA-seq data with drug treatments are too short for evaluation — avoid
  using this as a validated in-patient response predictor
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3391
tools:
- name: CaDRReS-Sc
  role: Recommender-system framework used to train drug response models on GDSC data
    and predict IC50/cell-death percentages
  repo: https://github.com/CSB5/CaDRReS-SC
- name: scDrug
  role: Wraps CaDRReS-Sc and PRISM models via drug_response_prediction.py to score
    cluster-wise IC50 and AUC sensitivity from scanpyobj.h5ad
  repo: https://github.com/ailabstw/scDrug
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.csbj.2022.11.055
    title: 'scDrug: From single-cell RNA-seq to drug response prediction'
  source_content_hash: sha256:27a08cc03ea7283fc34afd00ed1db1258f19f798f9c0033c347d0f7787761304
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CSB5/CaDRReS-SC
  tools:
  - CaDRReS-Sc
  - Python
  - scDrug
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3391
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# pharmacogenomic-drug-response-model-training

## Summary

Trains modified CaDRReS-Sc recommender-system models on GDSC IC50 and PRISM 19Q4 AUC response data with CCLE 21Q3 expression-derived kernel features, to predict drug response for cell lines or scRNA-seq clusters. Used in scDrug to reproduce the reported PRISM-vs-GDSC Pearson correlation of 0.615 and to score cluster-wise IC50/AUC sensitivity.

## When to use

Use when bulk drug response models must be trained or applied on GDSC/PRISM screening data with CCLE expression kernel features, e.g. reproducing cross-database correlation or scoring cluster-wise IC50/AUC predictions.

## When NOT to use

- Drug response prediction is limited to experimentally screened compounds — de novo sensitivity prediction for unscreened drugs is not supported
- Input is single-cell raw data needing preprocessing/clustering first; this skill starts from response tables plus CCLE expression or a completed scanpyobj.h5ad
- Available scRNA-seq data with drug treatments are too short for evaluation — avoid using this as a validated in-patient response predictor

## Inputs

- GDSC drug response data (measured IC50) for 226 drugs × 1074 cancer cell lines (from CaDRReS-Sc)
- PRISM Repurposing dataset version 19Q4 (AUC responses, 1448 drugs × 480 cell lines)
- CCLE expression data version 21Q3 from the DepMap Portal (1,379 cell lines, 19,177 genes)
- Scanpy Anndata object (scanpyobj.h5ad) with cluster GEPs when scoring single-cell clusters

## Outputs

- Trained modified CaDRReS-Sc models (10-dim latent space for GDSC; 140-dim for PRISM)
- IC50_prediction.csv and drug_kill_prediction.csv (--model GDSC) or AUC_prediction.csv (--model PRISM)
- prism_gdsc_correlation.csv containing the overall Pearson correlation between PRISM (1 − AUC) and GDSC (−log(IC50)) across shared drugs (expected ≈ 0.615)

## How to apply

Load the GDSC response data (measured IC50 for 226 drugs in 1074 cell lines, provided by CaDRReS-Sc) and the PRISM Repurposing dataset 19Q4 (1448 drugs against 480 cell lines). Transform response scales: convert GDSC IC50 to −log(IC50) and PRISM AUC to scaled 1 − AUC via D′ = 2.40D − 1.20. Build kernel features from CCLE 21Q3 expression: for the GDSC model use the 17,419 common genes across 1,018 cell lines; for the PRISM model select the 8,087 genes whose expression correlates with PRISM AUC at ≥ 0.2 absolute Pearson correlation, computing log2 expression fold-change per gene and Pearson-correlation cell–cell similarity. Train the modified CaDRReS-Sc models (drug bias only, no cell bias or overall mean term, L2 regularization) with a 10-dimensional latent space for GDSC and 140 for PRISM, learning rate 0.01, up to 100,000 epochs. Evaluate with drug-wise Spearman correlations between actual and predicted response, dropping drugs whose coefficient falls below the first-quartile coefficient.

## Related tools

- **CaDRReS-Sc** (Recommender-system framework used to train drug response models on GDSC data and predict IC50/cell-death percentages) — https://github.com/CSB5/CaDRReS-SC
- **scDrug** (Wraps CaDRReS-Sc and PRISM models via drug_response_prediction.py to score cluster-wise IC50 and AUC sensitivity from scanpyobj.h5ad) — https://github.com/ailabstw/scDrug

## Examples

```
python3 drug_response_prediction.py -i scanpyobj.h5ad -o ./output --model GDSC
```

## Evaluation signals

- Overall Pearson correlation between PRISM (1 − AUC) and GDSC (−log(IC50)) across shared drugs reproduces the reported value of 0.615 (written to prism_gdsc_correlation.csv)
- Drug-wise Spearman correlations between actual and predicted response are computed, and drugs below the first-quartile coefficient are dropped
- PRISM feature gene set contains 8,087 genes with ≥ 0.2 absolute Pearson correlation between expression and PRISM AUC; GDSC kernel uses 17,419 common genes across 1,018 cell lines
- Predicted sensitivity scores show significant negative correlation with post-treatment cell-proportion changes (Spearman −0.42, p = 0.04 in the idasanutlin validation)
- Output files match expected schemas: IC50_prediction.csv, drug_kill_prediction.csv (GDSC) or AUC_prediction.csv (PRISM)

## Limitations

- The prediction of drug response in scDrug is limited by experimentally screened compounds
- The available scRNA-seq data with drug treatments are short for evaluation
- De novo prediction of drug sensitivity is not yet implemented; extensive experimental validations of the pipeline are still necessary

## Evidence

- [methods] we used the response data (measured IC50) of 226 drugs in 1074 cancer cell lines, provided by CaDRReS-Sc from the GDSC database: "we used the response data (measured IC50) of 226 drugs in 1074 cancer cell lines, provided by CaDRReS-Sc from the GDSC database"
- [methods] We used 8,087 genes whose expressions were correlated with PRISM AUC with at least 0.2 absolute Pearson correlation coefficient as features genes: "We used 8,087 genes whose expressions were correlated with PRISM AUC with at least 0.2 absolute Pearson correlation coefficient as features genes"
- [results] the proportion changes were negatively correlated with the predicted sensitivity scores significantly (Spearman's correlation coefficient = −0.42 with p-value = 0.04): "the proportion changes were negatively correlated with the predicted sensitivity scores significantly (Spearman's correlation coefficient = −0.42 with p-value = 0.04)"
- [methods] we dropped the drugs whose drug-wise coefficient was lower than the first quartile coefficient.: "we dropped the drugs whose drug-wise coefficient was lower than the first quartile coefficient."
- [readme] reported clusterwise IC50 and cell death percentages to drugs in the GDSC database via CaDRReS-Sc: "reported clusterwise IC50 and cell death percentages to drugs in the GDSC database via [CaDRReS-Sc](https://github.com/CSB5/CaDRReS-SC)"
