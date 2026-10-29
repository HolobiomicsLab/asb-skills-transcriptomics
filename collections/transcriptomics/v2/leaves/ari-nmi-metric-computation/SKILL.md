---
name: ari-nmi-metric-computation
description: Use when evaluating clustering quality of an scRNA-seq imputation method
  (e.g. scGCL) where an h5 dataset provides both predicted clusters and true cell
  labels, and ARI/NMI scores must be reported per method-dataset pair.
when_to_use_negative:
- The data lacks ground-truth cell-type labels in the h5 file, since both ARI and
  NMI require a reference partition
- Evaluating gene imputation accuracy rather than clustering performance — the paper
  benchmarks these separately
- A single unreplicated run when sweeping graph hyperparameters, since averaged ARI/NMI
  over repeated runs is required for the sweep table
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: scGCL
  role: Produces the imputed matrix and predicted clusters whose ARI/NMI are computed
    against true labels
  repo: https://github.com/zehaoxiong123/scGCL
- name: scTAG
  role: Baseline imputation/clustering method scored with ARI/NMI in the sweep and
    comparison tables
- name: AutoClass
  role: Baseline method scored with ARI/NMI across the 14 scRNA-seq datasets
- name: GraphSCI
  role: Baseline method scored with ARI/NMI across the 14 scRNA-seq datasets
- name: MAGIC
  role: Baseline imputation method scored with ARI/NMI across the 14 scRNA-seq datasets
provenance:
  source_task_ids:
  - task_005
  - task_004
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btad098
    title: 'scGCL: an imputation method for scRNA-seq data based on graph contrastive
      learning'
  source_content_hash: sha256:9bef1eaaf3591ad5ef07ccfedce5c4b2f0bf4574a71723ae7744c1f1ded558ec
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# ari-nmi-metric-computation

## Summary

Computes Adjusted Rand Index (ARI) and Normalized Mutual Information (NMI) between predicted cell clusters and true cell-type labels to benchmark scRNA-seq imputation/clustering methods such as scGCL against baselines (AutoClass, GraphSCI, scTAG, MAGIC).

## When to use

Use when evaluating clustering quality of an scRNA-seq imputation method (e.g. scGCL) where an h5 dataset provides both predicted clusters and true cell labels, and ARI/NMI scores must be reported per method-dataset pair.

## When NOT to use

- The data lacks ground-truth cell-type labels in the h5 file, since both ARI and NMI require a reference partition
- Evaluating gene imputation accuracy rather than clustering performance — the paper benchmarks these separately
- A single unreplicated run when sweeping graph hyperparameters, since averaged ARI/NMI over repeated runs is required for the sweep table

## Inputs

- scRNA-seq dataset as an h5 file containing the expression matrix, true cell labels, and other metadata
- Predicted cluster assignments from scGCL (or baselines: AutoClass, GraphSCI, scTAG, MAGIC)
- Sweep parameters: k in {5,10,15,20,25} and distance measure in {city-block, cosine, euclidean, L2, manhattan}

## Outputs

- ARI and NMI values per method-dataset pair (e.g. scgcl_ad_metrics.csv)
- Averaged ARI/NMI sweep table over k–distance combinations
- Per-cluster cell-count comparison table reproducing Supplementary Fig. S2

## How to apply

Load the deposited scRNA-seq h5 file containing the expression matrix and true labels, run the clustering method (e.g. 'python main.py --dataset adam --task clustering --epochs 300' with the early-stopping criterion enabled), then compute ARI and NMI between predicted clusters and the true cell labels for every method-dataset pair. Repeat runs and average ARI/NMI over repetitions, especially when sweeping hyperparameters such as the KNN graph's k in {5,10,15,20,25} and distance measures (city-block, cosine, euclidean, L2, manhattan). Assemble the averaged values into a single metrics table across all method-dataset combinations, and for datasets like Quake_10x_Spleen also compare per-cluster cell counts against the true cell-type numbers. Accept the result when the table reproduces the reported ordering (scGCL outperforming baselines; scGCL on adam reaching ARI 0.62 / NMI 0.70).

## Related tools

- **scGCL** (Produces the imputed matrix and predicted clusters whose ARI/NMI are computed against true labels) — https://github.com/zehaoxiong123/scGCL
- **scTAG** (Baseline imputation/clustering method scored with ARI/NMI in the sweep and comparison tables)
- **AutoClass** (Baseline method scored with ARI/NMI across the 14 scRNA-seq datasets)
- **GraphSCI** (Baseline method scored with ARI/NMI across the 14 scRNA-seq datasets)
- **MAGIC** (Baseline imputation method scored with ARI/NMI across the 14 scRNA-seq datasets)

## Examples

```
python main.py --dataset adam --task clustering --epochs 300
```

## Evaluation signals

- Computed ARI/NMI for scGCL on the adam (Alzheimer's) dataset should reproduce ARI 0.62 and NMI 0.70
- Quake_10x_Spleen should yield scGCL ARI/NMI of 0.94/0.87, with per-cluster cell counts closest to the true cell-type numbers
- The k–distance sweep table should reproduce the finding that cosine distance with k=15 gives the optimal average ARI/NMI for both scGCL and scTAG
- Every method-dataset pair in the final table has both ARI and NMI averaged over repeated runs, consistent with Supplementary Fig. S2
- The imputed expression matrix exported before clustering has the same cell-by-gene dimensions as the input h5 expression matrix

## Limitations

- ARI/NMI scores depend on the hyperparameter choice — e.g. optimal clustering occurs only at k=15 with cosine distance, so other settings yield lower values
- Reported values are averages over repeated runs; single-run results may deviate from the published figures
- The clustering task on the adam dataset uses the published settings (300 epochs, early stopping); deviating settings may not reproduce ARI 0.62 / NMI 0.70

## Evidence

- [readme] scGCL outperforms existing state-of-the-art imputation methods in clustering performance and gene imputation on 14 real scRNA-seq datasets.: "scGCL outperforms existing state-of-the-art imputation methods in clustering performance and gene imputation on 14 real scRNA-seq datasets."
- [other] Compute ARI and NMI between predicted clusters and the true cell labels for every method-dataset pair.: [evidence span withheld — anchor status: not_found]
- [other] both scGCL and scTAG achieve their best clustering effect at k=15, with scGCL consistently outperforming scTAG across all tested settings: [evidence span withheld — anchor status: not_found]
