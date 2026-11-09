---
name: multi-omics-clustering-benchmarking
description: Use when benchmarking a multimodal single-cell clustering method against
  single-modality or concatenation-based ablations, or when reproducing ARI/NMI performance
  comparisons with statistical significance testing across multiple datasets.
when_to_use_negative:
- When the goal is joint embedding rather than clustering — the article notes existing
  joint-embedding methods 'are not designed and optimized for clustering', so embedding-only
  benchmarks answer a different question
- When comparing against competing multimodal methods like BREM-SC, CiteFuse, Seurat
  V4/WNN, Cobolt, scMM, or Specter — that is a separate comparison task from the ablation-vs-variant
  benchmarking covered here
- When only one modality and no paired multimodal data are available — the ablation
  design requires paired RNA + ADT/ATAC measurements
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_3170
tools:
- name: scMDC
  role: Multimodal deep clustering method under test and the source of its single-modality
    variants (scMDC-RNA/-ADT/-ATAC/-Concat)
- name: scDeepCluster
  role: Single-modal deep-learning baseline trained on all 13 datasets with matching
    preprocessing
- name: Scanpy
  role: Preprocessing and normalization of mRNA and ADT data (filter zero-count genes,
    size-factor normalization, log transform)
- name: PyTorch
  role: Training framework for the scMDC variant models
- name: scikit-learn
  role: Computation of ARI and NMI clustering performance metrics
- name: h5py
  role: Reading the h5-format input files containing X1, X2, and Y
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-022-35031-9
    title: Clustering of single-cell multi-omics data with a multimodal deep learning
      method
  source_content_hash: sha256:9b9f5d285c811f1595eff21cff2fbc74925193e2fea32396dc6738e192322973
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - NumPy
  - PyTorch
  - Scanpy
  - h5py
  - scDeepCluster
  - scMDC
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 15
---

# multi-omics-clustering-benchmarking

## Summary

Reproduce ablation and baseline benchmarking of the multimodal deep clustering method scMDC against its single-modality variants (scMDC-RNA/-ADT/-ATAC/-Concat) and scDeepCluster on CITE-seq and SMAGE-seq datasets, using ARI/NMI and one-sided paired t-tests to quantify the benefit of multimodal integration.

## When to use

Use when benchmarking a multimodal single-cell clustering method against single-modality or concatenation-based ablations, or when reproducing ARI/NMI performance comparisons with statistical significance testing across multiple datasets.

## When NOT to use

- When the goal is joint embedding rather than clustering — the article notes existing joint-embedding methods 'are not designed and optimized for clustering', so embedding-only benchmarks answer a different question
- When comparing against competing multimodal methods like BREM-SC, CiteFuse, Seurat V4/WNN, Cobolt, scMM, or Specter — that is a separate comparison task from the ablation-vs-variant benchmarking covered here
- When only one modality and no paired multimodal data are available — the ablation design requires paired RNA + ADT/ATAC measurements

## Inputs

- Preprocessed CITE-seq and SMAGE-seq count matrices in h5 format (X1 = RNA, X2 = ADT or ATAC, Y = cell type labels)
- 13 benchmark datasets including GSE100866 (CBMN), GSE128639 (BMNC/bmcite), GSE150599 (SLN208, SLN111), 10X PBMC, PBMC3K, PBMC10K, and E18 mouse brain

## Outputs

- Per-run clustering predictions (--prediction_file) for each method-dataset pair
- ARI and NMI metrics for every method-dataset combination
- fig7_performance_comparison.png (difference boxplots of performance)
- supp_table4_ttest_results.csv (one-sided paired t-test p-values)

## How to apply

Load preprocessed CITE-seq and SMAGE-seq count matrices from h5 files (X1 = RNA, X2 = ADT or ATAC, Y = labels), normalizing mRNA and ADT/ATAC separately with Scanpy (filter zero-count features, size-factor normalization, log transform). Train each scMDC variant with encoder layers [256, 64, 32, 16], 400 pretraining epochs, gamma 0.1, and up to 10000 clustering epochs; for the concatenated variant use sigma1 = 2.5 (RNA noise) and sigma2 = 1.5 (CITE-seq ADT) or 2.5 (SMAGE-seq ATAC). Train scDeepCluster on all 13 datasets with matching preprocessing as the deep-learning baseline. Compute ARI and NMI for every method-dataset pair, save per-run predictions via the --prediction_file option, render Fig. 7 difference boxplots as fig7_performance_comparison.png, and run one-sided paired t-tests across datasets writing p-values to supp_table4_ttest_results.csv.

## Related tools

- **scMDC** (Multimodal deep clustering method under test and the source of its single-modality variants (scMDC-RNA/-ADT/-ATAC/-Concat))
- **scDeepCluster** (Single-modal deep-learning baseline trained on all 13 datasets with matching preprocessing)
- **Scanpy** (Preprocessing and normalization of mRNA and ADT data (filter zero-count genes, size-factor normalization, log transform))
- **PyTorch** (Training framework for the scMDC variant models)
- **scikit-learn** (Computation of ARI and NMI clustering performance metrics)
- **h5py** (Reading the h5-format input files containing X1, X2, and Y)

## Examples

```
python main.py --dataset bmcite --X1 rna.h5 --X2 adt.h5 --prediction_file preds/scmdc_bmcite.csv  # then aggregate ARI/NMI and run one-sided paired t-tests across the 13 datasets
```

## Evaluation signals

- scMDC variants attain higher mean ARI and NMI than scDeepCluster and the single-modality variants on every CITE-seq and SMAGE-seq dataset
- One-sided paired t-test p-values reproduce the Supplementary Table 4 significance pattern; the only non-significant exception is scMDC-ATAC (P-value = 0.07) due to small SMAGE-seq sample size (n = 4)
- Difference boxplots (fig7_performance_comparison.png) show consistently positive scMDC-minus-baseline differences across datasets
- supp_table4_ttest_results.csv is produced with p-values for each scMDC-vs-baseline comparison
- Per-run prediction files are saved for every method-dataset pair via --prediction_file

## Limitations

- The number of clusters K must be specified by the user; no automatic estimation is provided
- All experiments require GPU hardware (Nvidia Tesla P100 16G), which may limit reproducibility on CPU-only systems
- With only n = 4 SMAGE-seq datasets, the one-sided paired t-test lacks power — the scMDC vs scMDC-ATAC comparison is not significant (P = 0.07) despite higher mean performance

## Evidence

- [results] Ablation significance pattern: [evidence span withheld — anchor status: not_found]
- [results] t-test exception: [evidence span withheld — anchor status: not_found]
- [discussion] Preprocessing protocol: "The raw CITE-seq data is preprocessed and normalized by the Python package SCANPY48. mRNA and ADT data are normalized separately but using the same method."
- [discussion] Filter step: "Specifically, the genes and ADTs with no count are filtered out."
- [discussion] K specification: "After pretraining, the users need to specify the number of clusters (K)."
- [discussion] GPU requirement: "All experiments of scMDC in this study are conducted on Nvidia Tesla P100 (16 G) GPU."
- [methods] h5 input prep: "1) Prepare the input data in h5 format. (See readme in 'dataset' folder)"
- [discussion] Pretraining epochs: "We pretrain the autoencoders for 400 epochs before entering the clustering stage."
