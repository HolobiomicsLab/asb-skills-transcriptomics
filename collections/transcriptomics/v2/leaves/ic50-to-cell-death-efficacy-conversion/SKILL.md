---
name: ic50-to-cell-death-efficacy-conversion
description: Use when you have per-cell or cluster-level IC50 predictions from CaDRReS-Sc
  GDSC pre-trained model and need to express drug kill efficacy as cell death percentages
  per cluster and drug.
when_to_use_negative:
- You want PRISM-based drug sensitivity predictions (AUC) rather than IC50/kill efficacy
  — use --model PRISM, which outputs AUC_prediction.csv instead
- Input is raw 10x mtx or CSV rather than a processed AnnData object from Single-Cell
  Data Analysis — cluster-wise IC50 requires the clustered scanpyobj.h5ad
- The compounds of interest were not experimentally screened in GDSC; scDrug drug
  response prediction is limited to experimentally screened compounds
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: scDrug
  role: Workflow orchestrating the IC50 prediction and conversion of IC50 into cell
    death percentages (drug kill efficacy) reported in IC50_prediction.csv and drug_kill_prediction.csv
  repo: https://github.com/ailabstw/scDrug
- name: CaDRReS-Sc
  role: Recommender-system framework whose GDSC pre-trained model predicts per-cell
    IC50 and cell death percentages from scRNA-seq expression, used for the cluster-wise
    aggregation
  repo: https://github.com/CSB5/CaDRReS-SC
- name: Scanpy
  role: Provides the AnnData (h5ad) input object and Louvain cluster labels over which
    IC50 predictions are averaged
- name: Python
  role: Execution environment for drug_response_prediction.py and the CaDRReS-Sc inference
    code
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.csbj.2022.11.055
    title: 'scDrug: From single-cell RNA-seq to drug response prediction'
  source_content_hash: sha256:c26f334a5c8daa0ba53a02fb1a3c3adf0487e023da4f159032afd8ea3a9ec81d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ailabstw/scDrug
  tools:
  - CaDRReS-Sc
  - Python
  - Scanpy
  - scDrug
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# ic50-to-cell-death-efficacy-conversion

## Summary

Converts predicted cluster-wise IC50 values (estimated via CaDRReS-Sc on scRNA-seq clusters) into cell death percentages that scDrug reports as drug kill efficacy for each predicted GDSC drug on each Louvain cluster.

## When to use

Use when you have per-cell or cluster-level IC50 predictions from CaDRReS-Sc GDSC pre-trained model and need to express drug kill efficacy as cell death percentages per cluster and drug.

## When NOT to use

- You want PRISM-based drug sensitivity predictions (AUC) rather than IC50/kill efficacy — use --model PRISM, which outputs AUC_prediction.csv instead
- Input is raw 10x mtx or CSV rather than a processed AnnData object from Single-Cell Data Analysis — cluster-wise IC50 requires the clustered scanpyobj.h5ad
- The compounds of interest were not experimentally screened in GDSC; scDrug drug response prediction is limited to experimentally screened compounds

## Inputs

- Scanpy AnnData object scanpyobj.h5ad from Single-Cell Data Analysis (input expression matrix)
- CaDRReS-Sc GDSC pre-trained model (10-dimensional latent space)
- GDSC drug response training data (226 drugs measured IC50 in 1074 cancer cell lines, via CaDRReS-Sc)

## Outputs

- IC50_prediction.csv (cluster-wise IC50 values)
- drug_kill_prediction.csv (per-cluster, per-drug kill efficacy as cell death percentages)

## How to apply

Load the Scanpy AnnData object scanpyobj.h5ad produced by Single-Cell Data Analysis, compute each cell's log2 fold change of gene expression with respect to the average expression of the AnnData object, and predict IC50 per cell with the CaDRReS-Sc GDSC pre-trained model (10-dimensional latent space). Aggregate per-cell IC50 predictions by taking the mean within each Louvain cluster to obtain cluster-wise IC50, then convert those IC50 values into cell death percentages, which scDrug reports as drug kill efficacy. Write cluster-wise IC50 to IC50_prediction.csv and per-cluster, per-drug kill efficacy to drug_kill_prediction.csv in the output directory. The rationale is that cell death percentage is a more interpretable efficacy measure than raw IC50 for ranking candidate drugs across heterogeneous tumor subpopulations.

## Related tools

- **scDrug** (Workflow orchestrating the IC50 prediction and conversion of IC50 into cell death percentages (drug kill efficacy) reported in IC50_prediction.csv and drug_kill_prediction.csv) — https://github.com/ailabstw/scDrug
- **CaDRReS-Sc** (Recommender-system framework whose GDSC pre-trained model predicts per-cell IC50 and cell death percentages from scRNA-seq expression, used for the cluster-wise aggregation) — https://github.com/CSB5/CaDRReS-SC
- **Scanpy** (Provides the AnnData (h5ad) input object and Louvain cluster labels over which IC50 predictions are averaged)
- **Python** (Execution environment for drug_response_prediction.py and the CaDRReS-Sc inference code)

## Examples

```
python3 drug_response_prediction.py -i scanpyobj.h5ad -o ./output -m GDSC
```

## Evaluation signals

- IC50_prediction.csv and drug_kill_prediction.csv both exist in the output directory when running with --model GDSC
- drug_kill_prediction.csv contains entries for every Louvain cluster and every predicted GDSC drug
- Kill efficacy values in drug_kill_prediction.csv are expressed as cell death percentages (bounded percentage scale, lower IC50 corresponding to higher death percentage)
- Cluster-wise IC50 equals the mean of per-cell IC50 predictions within each Louvain cluster
- Running with -c '1,3,8,9' restricts predictions to the specified clusters only

## Limitations

- Prediction is limited by experimentally screened compounds — drugs without GDSC screening data cannot be scored
- Lack of scRNA-seq data with drug treatments makes experimental evaluation of predicted kill efficacy difficult
- Drug response prediction depends on the Louvain clustering produced upstream; errors in clustering propagate into cluster-wise IC50 aggregation

## Evidence

- [intro] Drug Response Prediction estimated the half maximal inhibitory concentration (IC50) of cell clusters, and reported the cell death percentages as drug kill efficacy.: "Drug Response Prediction estimated the half maximal inhibitory concentration (IC50) of cell clusters, and reported the cell death percentages as drug kill efficacy."
- [readme] Reports IC50 as well as predicted cell death percentages for a specific drug dosage: [evidence span withheld — anchor status: external_doc]
- [readme] The output the prediction results are IC50_prediction.csv and drug_kill_prediction.csv while using parameter --model GDSC: "The output the prediction results are `IC50_prediction.csv` and `drug_kill_prediction.csv` while using parameter `--model GDSC`"
- [methods] we used the response data (measured IC50) of 226 drugs in 1074 cancer cell lines, provided by CaDRReS-Sc from the GDSC database: "we used the response data (measured IC50) of 226 drugs in 1074 cancer cell lines, provided by CaDRReS-Sc from the GDSC database"
- [discussion] The prediction of drug response in scDrug is limited by experimentally screened compounds: "The prediction of drug response in scDrug is limited by experimentally screened compounds"
