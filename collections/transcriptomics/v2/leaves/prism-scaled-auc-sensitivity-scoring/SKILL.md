---
name: prism-scaled-auc-sensitivity-scoring
description: Use when you have a Scanpy AnnData object (scanpyobj.h5ad) of drug-treated
  scRNA-seq samples with tumor clusters, and need PRISM-based per-cluster AUC sensitivity
  predictions to compare against observed cell proportion changes.
when_to_use_negative:
- You need IC50 or cell-kill percentages rather than AUC — use --model GDSC (CaDRReS-Sc
  GDSC pre-trained model) instead, which outputs IC50_prediction.csv and drug_kill_prediction.csv
- The target drug was never experimentally screened in PRISM; scDrug's response prediction
  is limited to experimentally screened compounds
- Input is raw scRNA-seq reads or an unclustered matrix — Single-Cell Data Analysis
  must first produce scanpyobj.h5ad
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: scDrug
  role: Provides the drug_response_prediction.py CLI implementing PRISM-based AUC
    sensitivity prediction on scanpyobj.h5ad
  repo: https://github.com/ailabstw/scDrug
- name: CaDRReS-Sc
  role: Recommender-system framework underlying the sensitivity prediction, using
    log2 fold-change features and a latent pharmacogenomic space
  repo: https://github.com/CSB5/CaDRReS-SC
- name: Scanpy
  role: Produces the scanpyobj.h5ad AnnData input and cluster annotations consumed
    by the prediction step
  repo: https://github.com/ailabstw/scDrug
- name: scipy.stats.spearmanr
  role: Computes the Spearman rank correlation between predicted sensitivity scores
    and observed cluster proportion changes for validation
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.csbj.2022.11.055
    title: 'scDrug: From single-cell RNA-seq to drug response prediction'
  source_content_hash: sha256:1b470f92ececff478ebe5841d0f804e058e3dc3916da14106ec0f9880688c4b0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ailabstw/scDrug
  tools:
  - CaDRReS-Sc
  - Python
  - Scanpy
  - scDrug
  - scipy.stats.spearmanr
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# prism-scaled-auc-sensitivity-scoring

## Summary

Predicts cluster-level drug sensitivity as scaled AUC values from scRNA-seq data using scDrug's Drug Response Prediction module with the pre-trained PRISM model (DepMap PRISM 19Q4, 140-dimensional latent space), built on the CaDRReS-Sc recommender-system framework. Sensitivity scores are then validated against observed cell-cluster proportion changes under drug treatment.

## When to use

Use when you have a Scanpy AnnData object (scanpyobj.h5ad) of drug-treated scRNA-seq samples with tumor clusters, and need PRISM-based per-cluster AUC sensitivity predictions to compare against observed cell proportion changes.

## When NOT to use

- You need IC50 or cell-kill percentages rather than AUC — use --model GDSC (CaDRReS-Sc GDSC pre-trained model) instead, which outputs IC50_prediction.csv and drug_kill_prediction.csv
- The target drug was never experimentally screened in PRISM; scDrug's response prediction is limited to experimentally screened compounds
- Input is raw scRNA-seq reads or an unclustered matrix — Single-Cell Data Analysis must first produce scanpyobj.h5ad

## Inputs

- Scanpy AnnData object (scanpyobj.h5ad) with clustered scRNA-seq data from drug-treated samples
- Cluster labels (e.g. adata.obs['louvain']) or a specified cluster list
- Pre-trained PRISM model (trained on DepMap PRISM Repurposing 19Q4 AUC responses)

## Outputs

- AUC_prediction.csv with a predicted scaled-AUC sensitivity score per cell cluster
- Per-cluster or per-cell-lineage sensitivity scores for correlation against observed proportion changes

## How to apply

Load the scanpyobj.h5ad AnnData produced by Single-Cell Data Analysis and run scDrug's drug_response_prediction.py with --model PRISM, which uses feature genes selected by |Pearson correlation| >= 0.2 with PRISM AUC (8,087 genes from the DepMap PRISM 19Q4 Repurposing dataset covering 1,448 drugs x 480 cell lines). The pre-trained PRISM model embeds samples into a 140-dimensional latent space and outputs learned variable-scaled AUC via the transformation D' = 240D - 120, writing per-cluster predictions to AUC_prediction.csv. For per-cluster prediction, compute log2 fold-change features relative to the average gene expression (or between a cluster and other clusters), as required by the CaDRReS-Sc framework. Validate the predictions by aggregating predicted sensitivity per cell lineage and computing the Spearman rank correlation against observed tumor-cluster proportion changes between treated and control samples (expected significant negative association, e.g. Spearman = -0.42, p = 0.04 for idasanutlin).

## Related tools

- **scDrug** (Provides the drug_response_prediction.py CLI implementing PRISM-based AUC sensitivity prediction on scanpyobj.h5ad) — https://github.com/ailabstw/scDrug
- **CaDRReS-Sc** (Recommender-system framework underlying the sensitivity prediction, using log2 fold-change features and a latent pharmacogenomic space) — https://github.com/CSB5/CaDRReS-SC
- **Scanpy** (Produces the scanpyobj.h5ad AnnData input and cluster annotations consumed by the prediction step) — https://github.com/ailabstw/scDrug
- **scipy.stats.spearmanr** (Computes the Spearman rank correlation between predicted sensitivity scores and observed cluster proportion changes for validation)

## Examples

```
python3 drug_response_prediction.py -i scanpyobj.h5ad -o ./output -m PRISM
```

## Evaluation signals

- AUC_prediction.csv exists and contains a predicted sensitivity column per cluster
- Predicted values fall in the scaled-AUC range implied by D' = 240D - 120
- Predicted sensitivity scores are negatively and significantly correlated (Spearman ~ -0.42, p ~ 0.04) with observed treated-vs-control cluster proportion changes, as reported for idasanutlin on 24 cell lines
- Feature gene set matches the 8,087 PRISM feature genes selected at |Pearson correlation| >= 0.2 with AUC
- Predictions cover all requested clusters (default 'All') from the input AnnData

## Limitations

- Prediction is limited to compounds experimentally screened in PRISM (DepMap PRISM 19Q4); de novo drug sensitivity prediction is not implemented
- Validation depends on scRNA-seq data with drug treatments, which are scarce for evaluation
- Extensive experimental validations of the pipeline are still needed; the scaled AUC is a model-derived surrogate, not a measured IC50

## Evidence

- [methods] Feature gene selection by AUC correlation: "We used 8,087 genes whose expressions were correlated with PRISM AUC with at least 0.2 absolute Pearson correlation coefficient as features genes"
- [readme] AUC output file with PRISM model: "The output the prediction results are `IC50_prediction.csv` and `drug_kill_prediction.csv` while using parameter `--model GDSC`, and `AUC_prediction.csv` whlie using parameter `--model PRISM`."
- [results] Validation against proportion changes: "the proportion changes were negatively correlated with the predicted sensitivity scores significantly (Spearman's correlation coefficient = −0.42 with p-value = 0.04)"
- [discussion] Scope limitation of the prediction: "The prediction of drug response in scDrug is limited by experimentally screened compounds"
