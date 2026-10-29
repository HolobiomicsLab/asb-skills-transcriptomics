---
name: drug-response-prediction-from-scrna-seq
description: Use when you have a clustered scRNA-seq AnnData object (scanpyobj.h5ad)
  and need clusterwise drug sensitivity predictions — e.g., IC50 from GDSC or AUC
  from PRISM — to rank drug candidates against tumor subpopulations.
when_to_use_negative:
- The input is raw scRNA-seq data (10x mtx or CSV) that has not yet been preprocessed
  and clustered — run single_cell_analysis.py first
- The drug of interest was not experimentally screened in GDSC or PRISM, since scDrug's
  drug response prediction is limited to experimentally screened compounds
- You need de novo drug sensitivity prediction for untested compounds, which is explicitly
  not yet implemented in scDrug
edam_operation: http://edamontology.org/operation_4001
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0208
- http://edamontology.org/topic_3320
tools:
- name: scDrug
  role: End-to-end workflow; drug_response_prediction.py wraps the prediction models
    and writes IC50/AUC/drug-kill CSV outputs
  repo: https://github.com/ailabstw/scDrug
- name: CaDRReS-Sc
  role: Recommender-system framework used for GDSC-based clusterwise IC50 and drug-kill
    prediction
  repo: https://github.com/CSB5/CaDRReS-SC
- name: Scanpy
  role: Produces the clustered AnnData (scanpyobj.h5ad) that serves as the prediction
    input
  repo: https://github.com/ailabstw/scDrug
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.csbj.2022.11.055
    title: 'scDrug: From single-cell RNA-seq to drug response prediction'
  source_content_hash: sha256:a65714ef10d7b560def0e6cd6ba29edca7a8ca37799e9eff7682e59e63874ec3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ailabstw/scDrug
  tools:
  - CaDRReS-Sc
  - Python
  - Scanpy
  - scDrug
  edam_operation: http://edamontology.org/operation_4001
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0208
  - http://edamontology.org/topic_3320
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# drug-response-prediction-from-scrna-seq

## Summary

Predicts drug sensitivity (IC50, cell death percentages, or PRISM AUC) per tumor cell cluster from a Scanpy AnnData object using CaDRReS-Sc (GDSC model) or a pre-trained PRISM model, enabling drug repurposing from scRNA-seq data.

## When to use

Use when you have a clustered scRNA-seq AnnData object (scanpyobj.h5ad) and need clusterwise drug sensitivity predictions — e.g., IC50 from GDSC or AUC from PRISM — to rank drug candidates against tumor subpopulations.

## When NOT to use

- The input is raw scRNA-seq data (10x mtx or CSV) that has not yet been preprocessed and clustered — run single_cell_analysis.py first
- The drug of interest was not experimentally screened in GDSC or PRISM, since scDrug's drug response prediction is limited to experimentally screened compounds
- You need de novo drug sensitivity prediction for untested compounds, which is explicitly not yet implemented in scDrug

## Inputs

- Scanpy AnnData object (scanpyobj.h5ad) with clustered scRNA-seq data
- Optional cluster specification (e.g., '1,3,8,9') to predict only selected clusters
- Choice of pre-trained model: GDSC (IC50 / drug kill) or PRISM-19Q4 (AUC)

## Outputs

- IC50_prediction.csv (clusterwise IC50, GDSC model)
- drug_kill_prediction.csv (cell death percentages, GDSC model)
- AUC_prediction.csv (predicted drug sensitivity AUC per cluster, PRISM model)

## How to apply

Run scDrug's drug_response_prediction.py on the scanpyobj.h5ad produced by Single-Cell Data Analysis. Choose --model GDSC to obtain IC50_prediction.csv and drug_kill_prediction.csv via CaDRReS-Sc, or --model PRISM to obtain AUC_prediction.csv from the PRISM Repurposing 19Q4 model (1448 drugs × 480 cell lines). For PRISM, 8,087 feature genes whose expression correlates with PRISM AUC at ≥0.2 absolute Pearson correlation are used, with log2 fold-change features relative to average gene expression; predictions are made per cell in a 140-dimensional latent space and averaged within each cluster (or predicted cluster-wise from cluster-vs-other log2 fold change). Optionally restrict prediction to specified clusters with --clusters. Validate by confirming the output CSV has a predicted sensitivity column per cluster and, when treated-vs-control scRNA-seq is available, by correlating predicted sensitivity with observed proportion changes (the idasanutlin case reproduced Spearman −0.42, p = 0.04).

## Related tools

- **scDrug** (End-to-end workflow; drug_response_prediction.py wraps the prediction models and writes IC50/AUC/drug-kill CSV outputs) — https://github.com/ailabstw/scDrug
- **CaDRReS-Sc** (Recommender-system framework used for GDSC-based clusterwise IC50 and drug-kill prediction) — https://github.com/CSB5/CaDRReS-SC
- **Scanpy** (Produces the clustered AnnData (scanpyobj.h5ad) that serves as the prediction input) — https://github.com/ailabstw/scDrug

## Examples

```
python3 drug_response_prediction.py -i scanpyobj.h5ad -o ./ --model PRISM
```

## Evaluation signals

- AUC_prediction.csv (or IC50_prediction.csv / drug_kill_prediction.csv) exists and contains a predicted sensitivity value for every requested cluster
- With treated-vs-control data, observed cell proportion changes correlate negatively with predicted sensitivity (reference result: Spearman −0.42, p = 0.04 for idasanutlin)
- For PRISM predictions, only the 8,087 feature genes (|Pearson r| ≥ 0.2 with PRISM AUC) drive the latent-space prediction
- Predicted values are on the expected model scale: GDSC IC50 values or PRISM AUC rescaled via D′ = 240D − 120
- Predictions are reproducible when restricted with --clusters, giving identical scores for the same cluster subset

## Limitations

- Drug response prediction in scDrug is limited to experimentally screened compounds in GDSC or PRISM
- Available scRNA-seq data with drug treatments are too sparse for broad evaluation; extensive experimental validations are still needed
- De novo prediction of drug sensitivity is not yet implemented
- Predictions are cluster-level averages and depend on clustering quality (resolution choice, sub-clustering) from the upstream Single-Cell Analysis step

## Evidence

- [intro] Drug Response Prediction estimated the half maximal inhibitory concentration (IC50) of cell clusters, and reported the cell death percentages as drug kill efficacy.: "Drug Response Prediction estimated the half maximal inhibitory concentration (IC50) of cell clusters, and reported the cell death percentages as drug kill efficacy."
- [methods] We used 8,087 genes whose expressions were correlated with PRISM AUC with at least 0.2 absolute Pearson correlation coefficient as features genes: "We used 8,087 genes whose expressions were correlated with PRISM AUC with at least 0.2 absolute Pearson correlation coefficient as features genes"
- [results] the proportion changes were negatively correlated with the predicted sensitivity scores significantly (Spearman's correlation coefficient = −0.42 with p-value = 0.04): "the proportion changes were negatively correlated with the predicted sensitivity scores significantly (Spearman's correlation coefficient = −0.42 with p-value = 0.04)"
- [discussion] The prediction of drug response in scDrug is limited by experimentally screened compounds: "The prediction of drug response in scDrug is limited by experimentally screened compounds"
- [readme] examined `scanpyobj.h5ad` generated in **Single-Cell Data Analysis**, reported clusterwise IC50 and cell death percentages to drugs in the GDSC database via [CaDRReS-Sc]: "examined `scanpyobj.h5ad` generated in **Single-Cell Data Analysis**, reported clusterwise IC50 and cell death percentages to drugs in the GDSC database via [CaDRReS-Sc]"
