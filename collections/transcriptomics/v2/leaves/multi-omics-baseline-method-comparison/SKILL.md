---
name: multi-omics-baseline-method-comparison
description: Use when you must show that a label-transfer method for unpaired scRNA-seq
  and scATAC-seq data is robust to reduced annotation budgets, by comparing average
  per-cell-type F1 against a semi-supervised baseline such as scJoint.
when_to_use_negative:
- The task is paired multi-omics integration where cross-omics pairing information
  is available (methods like scAI or MOFA+ are designed for that setting, not label-transfer
  comparison).
- Only one annotation fraction is needed and no baseline comparison is required —
  a single-method run suffices and the downsampled benchmark adds no value.
- Target data is protein (CITE-seq/ASAP-seq) rather than scATAC-seq gene activity;
  the scJoint protein-integration comparison is deferred to Supplementary Note 1,
  which is not covered by the provided document.
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0102
tools:
- name: scBridge
  role: Method under evaluation; iteratively identifies reliable scATAC-seq cells
    and transfers labels from annotated scRNA-seq, run via python main.py on a single
    GPU
  repo: https://github.com/XLearning-SCU/scBridge
- name: scJoint
  role: Semi-supervised baseline; integrates annotated scRNA-seq and unlabeled scATAC-seq
    via a neural network with KNN label transfer, configured in config.py
  repo: https://github.com/SydneyBioX/scJoint
- name: Signac
  role: R package used to transform scATAC-seq peaks into a gene activity matrix before
    cross-method comparison
  repo: https://github.com/stuart-lab/signac
- name: scikit-learn
  role: Provides Gaussian Mixture Model and metrics used for reliability modeling
    and per-cell-type F1 computation
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-023-41795-5
    title: scBridge embraces cell heterogeneity in single-cell RNA-seq and ATAC-seq
      data integration
  source_content_hash: sha256:cedc92dcf4f82ee0f190b95330664481bbc5fd0f7127e55c85bae551043da2bf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/XLearning-SCU/scBridge
  tools:
  - CUDA
  - PyTorch
  - Python
  - Signac
  - scBridge
  - scJoint
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# multi-omics-baseline-method-comparison

## Summary

Benchmark a proposed multi-omics integration method (scBridge) against a semi-supervised baseline (scJoint) on the same subsampled scRNA-seq/scATAC-seq inputs, quantifying label-transfer robustness via per-cell-type F1 under annotation downsampling.

## When to use

Use when you must show that a label-transfer method for unpaired scRNA-seq and scATAC-seq data is robust to reduced annotation budgets, by comparing average per-cell-type F1 against a semi-supervised baseline such as scJoint.

## When NOT to use

- The task is paired multi-omics integration where cross-omics pairing information is available (methods like scAI or MOFA+ are designed for that setting, not label-transfer comparison).
- Only one annotation fraction is needed and no baseline comparison is required — a single-method run suffices and the downsampled benchmark adds no value.
- Target data is protein (CITE-seq/ASAP-seq) rather than scATAC-seq gene activity; the scJoint protein-integration comparison is deferred to Supplementary Note 1, which is not covered by the provided document.

## Inputs

- 10x Multiome mouse kidney snRNA-seq count matrix as source h5ad with obs['CellType'] annotations
- 10x Multiome mouse kidney scATAC-seq peak matrix as target h5ad
- scATAC-seq gene activity matrix generated from peaks with Signac
- Subsampled annotation budgets of the source scRNA-seq cells (100%, 75%, 50%, 25%)
- Held-out target cell-type annotations for F1 evaluation

## Outputs

- kidney_label_transfer_f1.csv with average per-cell-type F1 for each method and annotation fraction
- scBridge label-transfer predictions on target scATAC-seq cells per fraction
- scJoint _knn_predictions.txt outputs per fraction
- Comparative robustness trend (e.g., scBridge 77.35%→77.08% vs scJoint 73.22%→59.36%)

## How to apply

Download the 10x Multiome mouse kidney snRNA-seq (source) and scATAC-seq (target) data from 10x Genomics and format them as h5ad AnnData files with cell-type labels in obs['CellType'] on the source. Convert scATAC-seq peaks into a gene activity matrix with Signac and retain only genes common to both modalities. Subsample the annotated scRNA-seq cells to 100%, 75%, 50%, and 25% annotation budgets while keeping the full unlabeled target data, then run scBridge (python main.py with defaults: 'Standard' source preprocessing, 'TFIDF' target preprocessing, reliability_threshold 0.95, batch_size 512, train_epoch 20, learning_rate 5e-4, random_seed 2023, max_iteration 20, early_stop_acc 0.99) and the official scJoint semi-supervised baseline (github.com/SydneyBioX/scJoint, configured via config.py) on identical inputs on a single GPU. Compute average per-cell-type label-transfer F1 on the target cells with scikit-learn metrics against held-out target annotations, assemble kidney_label_transfer_f1.csv, and verify the reported trend: scBridge drops only from 77.35% (full) to 77.08% (25%), while scJoint drops from 73.22% to 59.36% (Fig. 2g). The rationale is that iterative reliable-cell selection lets scBridge exploit heterogeneity, so it should degrade gracefully with fewer annotated cells than scJoint's semi-supervised neural network.

## Related tools

- **scBridge** (Method under evaluation; iteratively identifies reliable scATAC-seq cells and transfers labels from annotated scRNA-seq, run via python main.py on a single GPU) — https://github.com/XLearning-SCU/scBridge
- **scJoint** (Semi-supervised baseline; integrates annotated scRNA-seq and unlabeled scATAC-seq via a neural network with KNN label transfer, configured in config.py) — https://github.com/SydneyBioX/scJoint
- **Signac** (R package used to transform scATAC-seq peaks into a gene activity matrix before cross-method comparison) — https://github.com/stuart-lab/signac
- **scikit-learn** (Provides Gaussian Mixture Model and metrics used for reliability modeling and per-cell-type F1 computation)

## Examples

```
git clone https://github.com/XLearning-SCU/scBridge && python main.py --data_path ./data --source_data kidney_rna.h5ad --target_data kidney_atac.h5ad
```

## Evaluation signals

- At 100% annotation, scBridge reaches ~77.35% average F1 on the 10x Multiome mouse kidney target cells; at 25% annotation it stays at ~77.08% (change < 0.3 points).
- Under the same budgets, scJoint degrades from ~73.22% (full) to ~59.36% (25%) — a drop of roughly 14 points; reproduction should show scBridge > scJoint at every fraction.
- scBridge run terminates via early stopping when annotated-data classification accuracy reaches 99% (early_stop_acc 0.99), with max_iteration 20.
- F1 values are computed per cell type on target cells against held-out target annotations using scikit-learn metrics, then averaged; the CSV must contain one row per method × fraction.
- Both methods must be run on identical subsampled source inputs and the same common-gene target gene activity matrix to make the comparison fair.

## Limitations

- Protein-data integration results versus scJoint are in Supplementary Note 1, which is not included in the provided document.
- scJoint is trained with PyTorch 1.0.0/1.4.0 while scBridge uses PyTorch 1.12.1 with CUDA 11.4; environment differences may affect exact reproducibility of the baseline.
- Reported F1 values depend on the specific subsample of annotated cells; random_seed 2023 is used for scBridge but the baseline's seed is unset by default (scJoint config seed is None by default).
- scATAC-seq must first be converted to a gene activity matrix with tools such as Signac; comparing on raw peak matrices is out of scope for this skill.

## Evidence

- [other] scBridge maintains high label-transfer performance under annotation downsampling, achieving an average F1-score of 77.08% with only 25% annotated scRNA-seq cells (versus 77.35% on full data), while scJoint drops significantly from 73.22% to 59.36%.: [evidence span withheld — anchor status: not_found]
- [readme] For scATAC-seq peak data, it needs to be first transformed into a scATAC-seq activity matrix using tools such as Signac.: "For scATAC-seq peak data, it needs to be first transformed into a scATAC-seq activity matrix using tools such as Signac"
- [readme] scJoint's main function takes expression data in .npz format and cell type labels in .txt format.: [evidence span withheld — anchor status: not_found]
- [discussion] We adopted the Gaussian Mixture Model provided in the Scikit-learn Python package: "We adopted the Gaussian Mixture Model provided in the Scikit-learn Python package42, v.1.1.1."
- [discussion] we early stopped the network optimization when the cell classification accuracy of the annotated data reached 99%: "we early stopped the network optimization when the cell classification accuracy of the annotated data reached 99%"
