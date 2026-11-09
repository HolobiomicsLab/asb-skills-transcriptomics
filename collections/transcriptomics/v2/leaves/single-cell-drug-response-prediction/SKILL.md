---
name: single-cell-drug-response-prediction
description: Use when you have a clustered scRNA-seq AnnData object (scanpyobj.h5ad)
  and need cluster-wise drug sensitivity estimates (IC50) and cell-kill percentages
  for GDSC drugs.
when_to_use_negative:
- No Louvain clustering or AnnData object exists yet — run Single-Cell Data Analysis
  first to produce scanpyobj.h5ad
- The drug of interest was not experimentally screened in GDSC/PRISM — scDrug prediction
  is limited by experimentally screened compounds
- You need de novo prediction of drug sensitivity for unscreened compounds — this
  is explicitly not yet implemented
edam_topics:
- http://edamontology.org/topic_3336
- http://edamontology.org/topic_3308
tools:
- name: scDrug
  role: Workflow providing drug_response_prediction.py, which runs IC50/kill-efficacy
    prediction and writes IC50_prediction.csv and drug_kill_prediction.csv
  repo: https://github.com/ailabstw/scDrug
- name: CaDRReS-Sc
  role: Recommender-system framework with a GDSC pre-trained model (10-dimensional
    latent space) used to predict per-cell/cluster IC50 and cell death percentages
  repo: https://github.com/CSB5/CaDRReS-Sc
- name: Scanpy
  role: Produces the AnnData input (scanpyobj.h5ad) with Louvain cluster labels that
    the prediction aggregates over
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.csbj.2022.11.055
    title: 'scDrug: From single-cell RNA-seq to drug response prediction'
  source_content_hash: sha256:8dfc7e128eb811e55071d71d877e6dc7184bb982ffb58f3ff1398bde36569f3d
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
  - http://edamontology.org/topic_3336
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5263157894736842
n_claims: 19
---

# single-cell-drug-response-prediction

## Summary

Predicts drug response per Louvain cell cluster from a Scanpy AnnData object by estimating IC50 with the CaDRReS-Sc GDSC pre-trained model and converting it into cell death percentages reported as drug kill efficacy.

## When to use

Use when you have a clustered scRNA-seq AnnData object (scanpyobj.h5ad) and need cluster-wise drug sensitivity estimates (IC50) and cell-kill percentages for GDSC drugs.

## When NOT to use

- No Louvain clustering or AnnData object exists yet — run Single-Cell Data Analysis first to produce scanpyobj.h5ad
- The drug of interest was not experimentally screened in GDSC/PRISM — scDrug prediction is limited by experimentally screened compounds
- You need de novo prediction of drug sensitivity for unscreened compounds — this is explicitly not yet implemented

## Inputs

- Scanpy AnnData object (scanpyobj.h5ad) from Single-Cell Data Analysis, used as the input expression matrix
- Model choice: GDSC (226 drugs, 1074 cell lines, CaDRReS-Sc pre-trained) or PRISM (19Q4 Repurposing dataset)
- Optional cluster list, e.g. '1,3,8,9' (default 'All')

## Outputs

- IC50_prediction.csv (cluster-wise IC50 values)
- drug_kill_prediction.csv (per-cluster, per-drug cell death percentages / kill efficacy)
- AUC_prediction.csv (drug sensitivity AUC when using the PRISM model)

## How to apply

Load the Scanpy AnnData object scanpyobj.h5ad produced by Single-Cell Data Analysis as the input expression matrix. Compute the log2 fold change of each cell's gene expression relative to the average gene expression of the AnnData object, then predict per-cell IC50 with the CaDRReS-Sc GDSC pre-trained model built on a 10-dimensional latent pharmacogenomic space. Aggregate per-cell IC50 predictions by taking the mean within each Louvain cluster to obtain cluster-wise IC50, then convert these into cell death percentages that scDrug reports as drug kill efficacy for each predicted GDSC drug. Results are written to IC50_prediction.csv (cluster-wise IC50) and drug_kill_prediction.csv (per-cluster, per-drug kill efficacy); alternatively, PRISM AUC sensitivity can be predicted with --model PRISM producing AUC_prediction.csv.

## Related tools

- **scDrug** (Workflow providing drug_response_prediction.py, which runs IC50/kill-efficacy prediction and writes IC50_prediction.csv and drug_kill_prediction.csv) — https://github.com/ailabstw/scDrug
- **CaDRReS-Sc** (Recommender-system framework with a GDSC pre-trained model (10-dimensional latent space) used to predict per-cell/cluster IC50 and cell death percentages) — https://github.com/CSB5/CaDRReS-Sc
- **Scanpy** (Produces the AnnData input (scanpyobj.h5ad) with Louvain cluster labels that the prediction aggregates over)

## Examples

```
python3 drug_response_prediction.py -i scanpyobj.h5ad -o ./output --model GDSC
```

## Evaluation signals

- IC50_prediction.csv and drug_kill_prediction.csv exist in the output directory after a GDSC-model run
- Both output files contain entries for all Louvain clusters requested (or 'All' clusters) and all predicted GDSC drugs
- Drug kill efficacy in drug_kill_prediction.csv is expressed as cell death percentages (bounded 0-100%)
- Cluster-wise IC50 values equal the mean of the per-cell IC50 predictions within each cluster
- If --model PRISM is used instead, AUC_prediction.csv is produced rather than IC50/drug_kill files

## Limitations

- Drug response prediction is limited by experimentally screened compounds in the GDSC/PRISM training data
- Available scRNA-seq data with drug treatments are short for direct evaluation of the predictions
- De novo prediction of drug sensitivity is not implemented; extensive experimental validations of the pipeline are still necessary
- The PRISM model outputs drug sensitivity AUC (AUC_prediction.csv) rather than IC50/kill percentages

## Evidence

- [intro] Drug Response Prediction estimated the half maximal inhibitory concentration (IC50) of cell clusters, and reported the cell death percentages as drug kill efficacy.: "Drug Response Prediction estimated the half maximal inhibitory concentration (IC50) of cell clusters, and reported the cell death percentages as drug kill efficacy."
- [readme] The output the prediction results are IC50_prediction.csv and drug_kill_prediction.csv while using parameter --model GDSC: "The output the prediction results are `IC50_prediction.csv` and `drug_kill_prediction.csv` while using parameter `--model GDSC`"
- [methods] we used the response data (measured IC50) of 226 drugs in 1074 cancer cell lines, provided by CaDRReS-Sc from the GDSC database: "we used the response data (measured IC50) of 226 drugs in 1074 cancer cell lines, provided by CaDRReS-Sc from the GDSC database"
- [discussion] The prediction of drug response in scDrug is limited by experimentally screened compounds: "The prediction of drug response in scDrug is limited by experimentally screened compounds"
- [readme] Reports half-maximal inhibitory concentrations (IC50) as well as predicted cell death percentages for a specific drug dosage: [evidence span withheld — anchor status: external_doc]
