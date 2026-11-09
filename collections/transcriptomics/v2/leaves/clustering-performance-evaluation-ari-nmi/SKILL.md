---
name: clustering-performance-evaluation-ari-nmi
description: Use when you have per-cell cluster predictions from one or more single-cell
  clustering methods plus reference cell-type labels, and need to quantify and significance-test
  clustering accuracy (ARI/NMI).
when_to_use_negative:
- Clustering datasets where no reference cell-type labels are available, since ARI/NMI
  are external validity metrics requiring ground truth
- Evaluating embeddings or visualizations rather than discrete cell-cluster assignments
- Comparing methods on fewer than a handful of datasets, where the paired t-test is
  underpowered (as with SMAGE-seq n = 4)
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3474
tools:
- name: scMDC
  role: Multimodal deep clustering method whose cluster assignments are the primary
    predictions evaluated against ARI/NMI
  repo: https://github.com/xianglin226/scMDC
- name: scikit-learn
  role: Computes ARI and NMI between predicted cluster labels and reference labels
- name: PyTorch
  role: Trains scMDC variant models and scDeepCluster baseline whose clustering outputs
    are compared
- name: Scanpy
  role: Preprocessing/normalization of mRNA and ADT/ATAC data so all methods are evaluated
    on identically prepared inputs
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-022-35031-9
    title: Clustering of single-cell multi-omics data with a multimodal deep learning
      method
  source_content_hash: sha256:d4b77ded776148674ca6eaf00def8bde3f49f68b87dc2b3717f298eaffa2a4fe
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xianglin226/scMDC
  tools:
  - NumPy
  - PyTorch
  - Scanpy
  - h5py
  - scMDC
  - scikit-learn
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# clustering-performance-evaluation-ari-nmi

## Summary

Evaluates and statistically compares single-cell clustering methods (scMDC, its variants scMDC-RNA/-ADT/-ATAC/-Concat, scDeepCluster) by computing ARI and NMI per method-dataset pair on CITE-seq and SMAGE-seq data, then running one-sided paired t-tests across datasets.

## When to use

Use when you have per-cell cluster predictions from one or more single-cell clustering methods plus reference cell-type labels, and need to quantify and significance-test clustering accuracy (ARI/NMI).

## When NOT to use

- Clustering datasets where no reference cell-type labels are available, since ARI/NMI are external validity metrics requiring ground truth
- Evaluating embeddings or visualizations rather than discrete cell-cluster assignments
- Comparing methods on fewer than a handful of datasets, where the paired t-test is underpowered (as with SMAGE-seq n = 4)

## Inputs

- Preprocessed CITE-seq and SMAGE-seq count matrices in h5 format (X1 = RNA, X2 = ADT or ATAC, Y = reference cell-type labels)
- Per-run cluster prediction files (--prediction_file outputs) from each method-dataset pair

## Outputs

- ARI and NMI values for every method-dataset pair
- Difference boxplots of scMDC vs baseline performance (fig7_performance_comparison.png)
- One-sided paired t-test p-values (supp_table4_ttest_results.csv)

## How to apply

For each of the 13 datasets, train the methods under comparison (scMDC variants with encoder layers [256, 64, 32, 16], 400 pretraining epochs, gamma 0.1, max 10000 clustering epochs; scDeepCluster as the single-modal deep-learning baseline with matching Scanpy preprocessing) and save per-run predictions via the --prediction_file option. Compute ARI and NMI for every method-dataset pair using scikit-learn against the true labels (Y) stored in the h5 input files (X1 = RNA, X2 = ADT or ATAC). Aggregate per-dataset performance differences across datasets and render difference boxplots (Fig. 7). Run one-sided paired t-tests comparing scMDC against each baseline across datasets, writing p-values to supp_table4_ttest_results.csv. The statistical design matters: with small dataset counts the test loses power — the article reports the only non-significant comparison is scMDC vs scMDC-ATAC (P-value = 0.07) due to the small SMAGE-seq sample size (n = 4).

## Related tools

- **scMDC** (Multimodal deep clustering method whose cluster assignments are the primary predictions evaluated against ARI/NMI) — https://github.com/xianglin226/scMDC
- **scikit-learn** (Computes ARI and NMI between predicted cluster labels and reference labels)
- **PyTorch** (Trains scMDC variant models and scDeepCluster baseline whose clustering outputs are compared)
- **Scanpy** (Preprocessing/normalization of mRNA and ADT/ATAC data so all methods are evaluated on identically prepared inputs)

## Examples

```
from sklearn.metrics import adjusted_rand_score, normalized_mutual_info_score; ari = adjusted_rand_score(Y, pred); nmi = normalized_mutual_info_score(Y, pred)
```

## Evaluation signals

- scMDC variants should attain higher mean ARI/NMI than scDeepCluster and the single-modality variants on every CITE-seq and SMAGE-seq dataset
- One-sided paired t-test p-values should reproduce the Supplementary Table 4 significance pattern
- The sole non-significant comparison should be scMDC vs scMDC-ATAC (P = 0.07), explainable by SMAGE-seq n = 4
- Each method-dataset pair must have a saved prediction file whose labels have the same cell count as the reference Y
- ARI and NMI values should fall in [0, 1] and match reported values when recomputed from the prediction files

## Limitations

- The number of clusters K must be specified by the user; scMDC provides no automatic estimation
- All experiments were conducted on an Nvidia Tesla P100 (16 G) GPU, which may limit reproducibility on CPU-only systems
- Statistical significance is sensitive to dataset sample size: with SMAGE-seq n = 4, the scMDC vs scMDC-ATAC difference is not significant (P = 0.07)

## Evidence

- [results] scMDC outperforms all variant models: [evidence span withheld — anchor status: not_found]
- [results] P-value 0.07 for scMDC-ATAC: [evidence span withheld — anchor status: not_found]
- [methods] Compute ARI and NMI per pair: [evidence span withheld — anchor status: not_found]
- [abstract] scMDC outperforms existing methods (abstract): "Extensive simulation and real-data experiments reveal that scMDC outperforms existing single-cell single-modal and multimodal clustering methods on different single-cell multimodal datasets"
- [discussion] K must be specified: "After pretraining, the users need to specify the number of clusters (K)."
