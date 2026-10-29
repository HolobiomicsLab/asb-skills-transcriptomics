---
name: recommender-system-pharmacogenomic-inference
description: Use when a Scanpy AnnData object (scanpyobj.h5ad) from Single-Cell Data
  Analysis is available and you need cluster-wise IC50 and cell death percentage (drug
  kill efficacy) predictions for GDSC drugs.
when_to_use_negative:
- The input is raw 10x mtx or CSV scRNA-seq data that has not yet been clustered into
  a scanpyobj.h5ad by Single-Cell Data Analysis.
- The goal is PRISM drug sensitivity AUC prediction (--model PRISM, output AUC_prediction.csv)
  rather than GDSC IC50/kill-efficiency reporting.
- The drug of interest was not experimentally screened in the GDSC training data;
  scDrug's response prediction is limited to screened compounds.
edam_topics:
- http://edamontology.org/topic_0202
tools:
- name: scDrug
  role: Pipeline wrapper whose drug_response_prediction.py script runs the IC50/kill-efficiency
    prediction and writes the output CSVs
  repo: https://github.com/ailabstw/scDrug
- name: CaDRReS-Sc
  role: Recommender system framework with the GDSC pre-trained model used to predict
    per-cell and cluster-wise IC50 and cell death percentages
  repo: https://github.com/CSB5/CaDRReS-SC
- name: Scanpy
  role: Provides the AnnData (.h5ad) object format that serves as the input expression
    matrix
  repo: https://github.com/theislab/scanpy
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.csbj.2022.11.055
    title: 'scDrug: From single-cell RNA-seq to drug response prediction'
  source_content_hash: sha256:473522fdd0e7ecc23652a13f688792484a0b512ac34800591e978663629a256a
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
  - http://edamontology.org/topic_0202
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# recommender-system-pharmacogenomic-inference

## Summary

Predicts drug response of single-cell-defined tumor clusters by estimating IC50 with the CaDRReS-Sc recommender system (GDSC pre-trained model) and converting predictions to cell death percentages reported as drug kill efficacy.

## When to use

Use when a Scanpy AnnData object (scanpyobj.h5ad) from Single-Cell Data Analysis is available and you need cluster-wise IC50 and cell death percentage (drug kill efficacy) predictions for GDSC drugs.

## When NOT to use

- The input is raw 10x mtx or CSV scRNA-seq data that has not yet been clustered into a scanpyobj.h5ad by Single-Cell Data Analysis.
- The goal is PRISM drug sensitivity AUC prediction (--model PRISM, output AUC_prediction.csv) rather than GDSC IC50/kill-efficiency reporting.
- The drug of interest was not experimentally screened in the GDSC training data; scDrug's response prediction is limited to screened compounds.

## Inputs

- Scanpy AnnData object (scanpyobj.h5ad) from Single-Cell Data Analysis
- GDSC pre-trained CaDRReS-Sc model (10-dimensional latent space, 226 drugs x 1074 cell lines IC50 training data)

## Outputs

- IC50_prediction.csv (cluster-wise predicted IC50 values)
- drug_kill_prediction.csv (per-cluster, per-drug kill efficacy as cell death percentages)

## How to apply

Load the Scanpy AnnData object scanpyobj.h5ad produced by Single-Cell Data Analysis as the expression matrix input to drug_response_prediction.py with --model GDSC. Compute the log2 fold change of each cell's expression relative to the average expression values, and predict per-cell IC50 with the CaDRReS-Sc GDSC pre-trained model (10-dimensional latent pharmacogenomic space). Aggregate per-cell IC50 predictions by taking the mean IC50 within each Louvain cluster to obtain cluster-wise IC50, then convert these values into cell death percentages, which scDrug reports as drug kill efficacy per drug per cluster. Rationale: the recommender-system framework projects cells and drugs into a shared latent space calibrated on 226 GDSC drugs screened in 1074 cancer cell lines, making it robust to expression measurements across diverse platforms.

## Related tools

- **scDrug** (Pipeline wrapper whose drug_response_prediction.py script runs the IC50/kill-efficiency prediction and writes the output CSVs) — https://github.com/ailabstw/scDrug
- **CaDRReS-Sc** (Recommender system framework with the GDSC pre-trained model used to predict per-cell and cluster-wise IC50 and cell death percentages) — https://github.com/CSB5/CaDRReS-SC
- **Scanpy** (Provides the AnnData (.h5ad) object format that serves as the input expression matrix) — https://github.com/theislab/scanpy

## Examples

```
python3 drug_response_prediction.py -i scanpyobj.h5ad -m GDSC -o ./output
```

## Evaluation signals

- IC50_prediction.csv and drug_kill_prediction.csv exist in the output directory when --model GDSC is used.
- Both CSVs contain entries for all Louvain clusters and all predicted GDSC drugs (up to --n_drugs).
- Drug kill efficacy values are expressed as cell death percentages, not raw IC50 units.
- Cluster-wise IC50 equals the mean of per-cell IC50 predictions within each cluster.
- Per-cell IC50 predictions derive from log2 fold-change inputs to the GDSC pre-trained model, consistent with CaDRReS-Sc's reporting of IC50 and predicted cell death percentages.

## Limitations

- Prediction of drug response in scDrug is limited to experimentally screened compounds in the training database (GDSC or PRISM).
- Available scRNA-seq data with drug treatments are too short for external evaluation of the predictions.
- De novo prediction of drug sensitivity is not implemented, and extensive experimental validation of the pipeline is still needed.

## Evidence

- [intro] Drug Response Prediction estimated the half maximal inhibitory concentration (IC50) of cell clusters, and reported the cell death percentages as drug kill efficacy.: "**Drug Response Prediction** estimated the half maximal inhibitory concentration (IC50) of cell clusters, and reported the cell death percentages as drug kill efficacy."
- [readme] via CaDRReS-Sc (a recommender system framework for in silico drug response prediction): "via [CaDRReS-Sc](https://github.com/CSB5/CaDRReS-SC) (a recommender system framework for *in silico* drug response prediction)"
- [methods] we used the response data (measured IC50) of 226 drugs in 1074 cancer cell lines, provided by CaDRReS-Sc from the GDSC database: "we used the response data (measured IC50) of 226 drugs in 1074 cancer cell lines, provided by CaDRReS-Sc from the GDSC database"
- [readme] Reports half-maximal inhibitory concentrations (IC50) as well as predicted cell death percentages for a specific drug dosage: [evidence span withheld — anchor status: external_doc]
- [readme] The output the prediction results are `IC50_prediction.csv` and `drug_kill_prediction.csv` while using parameter `--model GDSC`: "The output the prediction results are `IC50_prediction.csv` and `drug_kill_prediction.csv` while using parameter `--model GDSC`"
