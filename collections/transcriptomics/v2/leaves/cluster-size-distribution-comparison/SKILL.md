---
name: cluster-size-distribution-comparison
description: 'Use when comparing imputation-plus-clustering pipelines on labeled scRNA-seq
  data and ARI/NMI alone is insufficient — e.g., checking whether predicted per-cluster
  cell counts match true cell-type proportions (Quake_10x_Spleen: scGCL ARI 0.94 /
  NMI 0.87).'
when_to_use_negative:
- The dataset has no deposited true cell labels, so per-cluster counts cannot be compared
  to ground truth
- The analysis goal is differential expression or gene imputation quality rather than
  clustering-faithfulness assessment
- Only one method's clustering is available with no baseline to compare the cluster-size
  distribution against
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scGCL
  role: Produces the imputed representation and clustering whose per-cluster cell
    counts are compared to true cell-type numbers
  repo: https://github.com/zehaoxiong123/scGCL
- name: AutoClass
  role: Baseline imputation/clustering method whose cluster-size distribution is compared
    against scGCL
- name: GraphSCI
  role: Baseline imputation/clustering method whose cluster-size distribution is compared
    against scGCL
- name: scTAG
  role: Baseline imputation/clustering method whose cluster-size distribution is compared
    against scGCL
- name: MAGIC
  role: Baseline imputation/clustering method whose cluster-size distribution is compared
    against scGCL
- name: Python
  role: Runtime environment (3.8.8) for loading h5 datasets, running clustering, and
    assembling the metrics table
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad098
    title: 'scGCL: an imputation method for scRNA-seq data based on graph contrastive
      learning'
  source_content_hash: sha256:9eb4682d9c904113fd53d69aaf401a729f8da3b5e1f3f95e5147091276f8dd30
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zehaoxiong123/scGCL
  tools:
  - AutoClass
  - GraphSCI
  - MAGIC
  - PyTorch
  - Python
  - faiss-cpu
  - scGCL
  - scTAG
  - torch-cluster
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# cluster-size-distribution-comparison

## Summary

Validates clustering quality on scRNA-seq data by comparing the number of cells assigned to each predicted cluster against the true cell-type counts, as done for scGCL versus AutoClass, GraphSCI, scTAG, and MAGIC on the Quake_10x_Spleen dataset.

## When to use

Use when comparing imputation-plus-clustering pipelines on labeled scRNA-seq data and ARI/NMI alone is insufficient — e.g., checking whether predicted per-cluster cell counts match true cell-type proportions (Quake_10x_Spleen: scGCL ARI 0.94 / NMI 0.87).

## When NOT to use

- The dataset has no deposited true cell labels, so per-cluster counts cannot be compared to ground truth
- The analysis goal is differential expression or gene imputation quality rather than clustering-faithfulness assessment
- Only one method's clustering is available with no baseline to compare the cluster-size distribution against

## Inputs

- h5 file containing the scRNA-seq expression matrix, true cell labels, and other metadata (e.g., Quake_10x_Spleen)
- Predicted cluster assignments from each method (scGCL, AutoClass, GraphSCI, scTAG, MAGIC)
- ARI and NMI values per method-dataset pair

## Outputs

- Per-cluster cell counts for each method versus true per-cell-type cell counts
- Combined ARI/NMI metrics table across the 14 datasets reproducing Supplementary Fig. S2

## How to apply

After running each method's clustering on an h5 scRNA-seq dataset (expression matrix plus true labels), group the predicted cluster assignments and count cells per predicted cluster, then tabulate the true per-cell-type counts from the deposited labels. Compare these count distributions side by side across methods (here: scGCL vs AutoClass, GraphSCI, scTAG, MAGIC) to see which method's cluster sizes most closely track the ground truth. For the scGCL benchmark, this comparison on Quake_10x_Spleen complements ARI/NMI metrics and showed scGCL's per-cluster cell counts were closest to the true cell-type numbers. Assemble the per-cluster counts together with the ARI/NMI values into a single metrics table (reproducing Supplementary Fig. S2). Accept the comparison when the winning method's ARI/NMI consistently exceed the baselines and its cluster-size distribution best matches the true label distribution.

## Related tools

- **scGCL** (Produces the imputed representation and clustering whose per-cluster cell counts are compared to true cell-type numbers) — https://github.com/zehaoxiong123/scGCL
- **AutoClass** (Baseline imputation/clustering method whose cluster-size distribution is compared against scGCL)
- **GraphSCI** (Baseline imputation/clustering method whose cluster-size distribution is compared against scGCL)
- **scTAG** (Baseline imputation/clustering method whose cluster-size distribution is compared against scGCL)
- **MAGIC** (Baseline imputation/clustering method whose cluster-size distribution is compared against scGCL)
- **Python** (Runtime environment (3.8.8) for loading h5 datasets, running clustering, and assembling the metrics table)

## Examples

```
python main.py --dataset Quake_10x_Spleen --task clustering --epochs 300
```

## Evaluation signals

- scGCL's ARI/NMI on Quake_10x_Spleen reach 0.94 and 0.87, exceeding all four baselines
- scGCL's number of cells assigned per cluster is closest to the true cell-type numbers on Quake_10x_Spleen
- ARI and NMI are computed between predicted clusters and true cell labels for every method-dataset pair across all 14 datasets
- scGCL's ARI/NMI values are consistent with the finding that it outperforms the four imputation baselines across the 14 scRNA-seq datasets (average ARI 0.82 / NMI 0.80)
- The assembled table reproduces Supplementary Fig. S2

## Limitations

- Cluster-size fidelity is only checkable where ground-truth cell-type labels are deposited in the h5 files
- The comparison is meaningful as a supplement to, not a replacement for, ARI/NMI: the validation criterion requires both consistent ARI/NMI superiority and closest per-cluster counts
- Results depend on scGCL runtime settings (e.g., 300 epochs, early stopping criterion enabled) matching the published configuration

## Evidence

- [results] On the Quake_10x_Spleen dataset, scGCL's clustering results far outperform all four comparison methods, with ARI and NMI reaching 0.94 and 0.87, and scGCL's per-cluster cell counts are closest to the true cell-type numbers.: [evidence span withheld — anchor status: not_found]
- [methods] Run scGCL command line: "You can run the scGCL from the command line:"
- [readme] README usage: "A h5 file. Contains a matrix of scRNA-seq expression values,true labels, and other information."
- [readme] README task argument: "Downstream task. Supported tasks are: node, clustering, similarity"
- [abstract] Overall claim on 14 datasets: "scGCL outperforms existing state-of-the-art imputation methods in clustering performance and gene imputation on 14 scRNA-seq datasets"
