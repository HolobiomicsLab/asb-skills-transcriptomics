---
name: cluster-level-sensitivity-aggregation
description: Use when scRNA-seq samples from drug-treated and control conditions are
  clustered (scanpyobj.h5ad) and you need cluster/lineage-level predicted drug sensitivity
  to compare with observed cell-proportion changes.
when_to_use_negative:
- You need de novo drug sensitivity prediction for compounds not present in the experimentally
  screened GDSC/PRISM datasets — the article notes this is 'limited by experimentally
  screened compounds'.
- No treated/control replicate design exists, so cluster proportion changes between
  conditions cannot be computed.
- The input is raw sequencing data rather than a pre-clustered AnnData (scanpyobj.h5ad)
  — run single_cell_analysis.py first.
edam_operation: http://edamontology.org/operation_2929
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scDrug
  role: Provides drug_response_prediction.py with the --model PRISM option that produces
    AUC_prediction.csv from the cluster/cell-level input
  repo: https://github.com/ailabstw/scDrug
- name: CaDRReS-Sc
  role: Recommender-system framework underlying the predictions; consumes log2 fold-change
    features to score cluster sensitivity
  repo: https://github.com/CSB5/CaDRReS-SC
- name: Scanpy
  role: Supplies the AnnData object (scanpyobj.h5ad) and cluster labels that the sensitivity
    predictions are aggregated over
- name: Python
  role: Environment in which predictions, aggregation, and scipy.stats.spearmanr validation
    are executed
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.csbj.2022.11.055
    title: 'scDrug: From single-cell RNA-seq to drug response prediction'
  source_content_hash: sha256:9adc4cc3a3cc40a41ef3dcf6e4bcc135a6f186749c7a450b1f9a39ff8ed535b5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ailabstw/scDrug
  tools:
  - CaDRReS-Sc
  - Python
  - Scanpy
  - scDrug
  edam_operation: http://edamontology.org/operation_2929
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# Cluster-level sensitivity aggregation

## Summary

Aggregates single-cell drug response predictions (e.g. PRISM AUC from scDrug's CaDRReS-Sc-based prediction) to cluster or lineage level, then validates them against observed treatment-induced cluster proportion changes via Spearman rank correlation.

## When to use

Use when scRNA-seq samples from drug-treated and control conditions are clustered (scanpyobj.h5ad) and you need cluster/lineage-level predicted drug sensitivity to compare with observed cell-proportion changes.

## When NOT to use

- You need de novo drug sensitivity prediction for compounds not present in the experimentally screened GDSC/PRISM datasets — the article notes this is 'limited by experimentally screened compounds'.
- No treated/control replicate design exists, so cluster proportion changes between conditions cannot be computed.
- The input is raw sequencing data rather than a pre-clustered AnnData (scanpyobj.h5ad) — run single_cell_analysis.py first.

## Inputs

- scanpyobj.h5ad AnnData object of drug-treated and control scRNA-seq samples
- Pre-trained PRISM model (PRISM Repurposing dataset 19Q4, 1448 drugs x 480 cell lines) from scDrug
- Cluster assignments (e.g. 'louvain' column) and per-cluster/lineage membership
- Observed per-cluster proportions in treated vs control samples

## Outputs

- AUC_prediction.csv with a predicted sensitivity column per cluster
- Per-lineage aggregated sensitivity scores
- Spearman correlation coefficient and p-value (expected: −0.42, p = 0.04)

## How to apply

Load the scanpyobj.h5ad AnnData of the treated samples and compute log2 fold-change features relative to the average gene expression, as required by the CaDRReS-Sc prediction framework. Apply the pre-trained PRISM model (140-dimensional latent space, learned variable scaled 1-AUC via D′ = 240D − 120) with drug_response_prediction.py --model PRISM to write AUC_prediction.csv, predicting sensitivity for each cell and averaging within each cluster, or predicting cluster-wise directly from the log2 fold change between a cluster and other clusters. Aggregate predicted idasanutlin sensitivity by cell lineage and compute the Spearman rank correlation between per-lineage predicted sensitivity and observed changes in tumor-cluster proportions between treated and control samples using scipy.stats.spearmanr. A valid result reproduces the reported significant negative association (Spearman −0.42, p = 0.04), showing that clusters predicted to be sensitive are depleted after treatment.

## Related tools

- **scDrug** (Provides drug_response_prediction.py with the --model PRISM option that produces AUC_prediction.csv from the cluster/cell-level input) — https://github.com/ailabstw/scDrug
- **CaDRReS-Sc** (Recommender-system framework underlying the predictions; consumes log2 fold-change features to score cluster sensitivity) — https://github.com/CSB5/CaDRReS-SC
- **Scanpy** (Supplies the AnnData object (scanpyobj.h5ad) and cluster labels that the sensitivity predictions are aggregated over)
- **Python** (Environment in which predictions, aggregation, and scipy.stats.spearmanr validation are executed)

## Examples

```
python3 drug_response_prediction.py -i scanpyobj.h5ad -m PRISM -o ./
```

## Evaluation signals

- AUC_prediction.csv exists and contains a predicted sensitivity value for every cluster
- Spearman correlation between per-lineage predicted sensitivity and observed proportion changes is significantly negative, reproducing −0.42 (p = 0.04)
- Predicted AUC values fall on the scaled 1-AUC range implied by D′ = 240D − 120 (i.e. back-transformable to [0,1])
- Log2 fold-change feature matrix is computed against average gene expression values as required by CaDRReS-Sc, not raw counts
- Clusters predicted most sensitive show the largest depletion in treated vs control samples (consistency check on rank ordering)

## Limitations

- Drug response prediction is limited to experimentally screened compounds; de novo sensitivity prediction is not yet implemented.
- Available scRNA-seq data with drug treatments are scarce, restricting evaluation of the pipeline.
- Extensive experimental validations of the pipeline are still necessary.

## Evidence

- [readme] reported clusterwise IC50 and cell death percentages to drugs in the GDSC database via [CaDRReS-Sc](https://github.com/CSB5/CaDRReS-SC): "reported clusterwise IC50 and cell death percentages to drugs in the GDSC database via [CaDRReS-Sc](https://github.com/CSB5/CaDRReS-SC)"
- [readme] AUC_prediction.csv via --model PRISM: "and `AUC_prediction.csv` whlie using parameter `--model PRISM`"
- [methods] PRISM AUC feature genes: [evidence span withheld — anchor status: not_found]
- [results] Validation correlation: "the proportion changes were negatively correlated with the predicted sensitivity scores significantly (Spearman's correlation coefficient = −0.42 with p-value = 0.04)"
- [discussion] Scope limitation: "The prediction of drug response in scDrug is limited by experimentally screened compounds"
