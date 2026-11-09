---
name: single-cell-clustering-benchmarking
description: Use when comparing clustering performance of multiple single-cell (multimodal
  or single-modal) methods on datasets with known ground-truth cell-type labels, in
  single-batch or multi-batch settings.
when_to_use_negative:
- The datasets lack ground-truth cell-type labels, so external-validation metrics
  (AMI, NMI, ARI) cannot be computed
- The goal is to train/apply scMDC to a single dataset rather than to comparatively
  evaluate multiple clustering methods
- The data is not single-cell multimodal data (CITE-seq or SMAGE-seq) and the user
  instead needs batch correction or differential expression analysis
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scMDC
  role: Method under benchmark test; multimodal deep clustering model run with CITE-Seq
    or SMAGE-seq defaults
  repo: https://github.com/xianglin226/scMDC
- name: BREM-SC
  role: Competing CITE-seq clustering method (hierarchical Bayesian mixture model,
    v0.2.0)
  repo: https://github.com/tarot0410/BREMSC
- name: CiteFuse
  role: Competing CITE-seq clustering method (similarity network fusion, v1.0.0)
  repo: https://github.com/SydneyBioX/CiteFuse
- name: Specter
  role: Competing method; landmark-based spectral clustering for single-cell and multimodal
    data
  repo: https://github.com/canzarlab/Specter
- name: Seurat V4
  role: Competing method; WNN procedure for multi-omics clustering (v4.0.4)
  repo: https://github.com/satijalab/seurat
- name: SCVIS
  role: Competing single-modal deep embedding method (v0.1.0)
  repo: https://github.com/shahcompbio/scvis
- name: IDEC
  role: Competing deep embedded clustering baseline
  repo: https://github.com/XifengGuo/IDEC
- name: SC3
  role: Competing single-modal spectral clustering method (v1.21.0)
  repo: https://github.com/hemberg-lab/SC3
- name: Tscan
  role: Competing single-modal method using PCA plus Gaussian mixture model clustering
    (v1.31.0)
  repo: https://github.com/zji90/TSCAN
- name: Cobolt
  role: Competing SMAGE-seq multimodal method (v1.0.0)
  repo: https://github.com/epurdom/cobolt
- name: scMM
  role: Competing SMAGE-seq multimodal deep generative method (v1.0.0)
  repo: https://github.com/kodaim1115/scMM
- name: Scanpy
  role: Preprocessing/normalization of mRNA and ADT data before benchmarking
- name: SKlearn
  role: Computation of AMI, NMI, and ARI external-validation metrics
provenance:
  source_task_ids:
  - task_001
  - task_002
  source_papers:
  - doi: 10.1038/s41467-022-35031-9
    title: Clustering of single-cell multi-omics data with a multimodal deep learning
      method
  source_content_hash: sha256:735345482852e3fdbbedf3dcb19c41fde7a0ffa29526d037b03ba9d6cb0cc989
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xianglin226/scMDC
  tools:
  - BREM-SC
  - CiteFuse
  - Cobolt
  - IDEC
  - SC3
  - SCVIS
  - SKlearn
  - Scanpy
  - Seurat V4
  - Specter
  - TotalVI
  - Tscan
  - scMDC
  - scMM
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9411764705882353
n_claims: 17
---

# single-cell-clustering-benchmarking

## Summary

Benchmark single-cell multimodal clustering methods (e.g. scMDC vs. 10 competing methods) on CITE-seq and SMAGE-seq datasets by computing AMI, NMI, and ARI against ground-truth cell-type labels and testing significance with one-sided paired t-tests across datasets.

## When to use

Use when comparing clustering performance of multiple single-cell (multimodal or single-modal) methods on datasets with known ground-truth cell-type labels, in single-batch or multi-batch settings.

## When NOT to use

- The datasets lack ground-truth cell-type labels, so external-validation metrics (AMI, NMI, ARI) cannot be computed
- The goal is to train/apply scMDC to a single dataset rather than to comparatively evaluate multiple clustering methods
- The data is not single-cell multimodal data (CITE-seq or SMAGE-seq) and the user instead needs batch correction or differential expression analysis

## Inputs

- h5 files containing X1 (mRNA counts), X2 (ADT or ATAC-derived gene activity counts), Y ground-truth cell-type labels, and a Batch indicator
- CITE-seq datasets: GSE100866 CBMN, GSE128639 BMNC, GSE150599 SLN208 and SLN111, 10X PBMC, plus two multi-batch datasets
- SMAGE-seq datasets: PBMC3K, PBMC10K, E18 mouse brain, multi-batch PBMC13K
- Clustering predictions from each method on identically preprocessed data

## Outputs

- results_table.csv with AMI/NMI/ARI scores, per-dataset ranks, and one-sided paired t-test p-values for every method
- smage_seq_benchmark_results.csv with NMI/AMI/ARI, ranks, and paired t-test p-values for all five methods on all datasets
- Prediction files from each method (e.g. scMDC cluster assignments, potentially with estimated K)

## How to apply

Prepare each dataset in h5 format with X1 (mRNA), X2 (ADT or ATAC), Y ground-truth labels, and a Batch indicator, normalizing mRNA and ADT/ATAC separately with Scanpy. Run the method under test (scMDC with CITE-Seq defaults: encodeLayer [256,64,32,16], pretrain_epochs 400, gamma 0.1, tol 0.001, sigma1 2.5, sigma2 1.5, phi1/phi2 0.001; or SMAGE-seq defaults: encodeLayer [256,128,64], sigma2 2.5, phi 0.005) on GPU, using run_scMDC_batch.py for multi-batch data. Run competitors (BREM-SC, CiteFuse, Specter, Seurat V4 WNN, SCVIS, TotalVI, SC3, Tscan, IDEC, K-means; or Cobolt, scMM for SMAGE-seq) with their published default settings on the same preprocessed data. Compute AMI, NMI, and ARI of every method's clustering against ground-truth labels with scikit-learn external-validation metrics, rank methods per dataset and per metric, then perform one-sided paired t-tests of the focal method's scores versus each competitor across datasets. Compile scores, ranks, and p-values into a results table (e.g. results_table.csv / smage_seq_benchmark_results.csv) reproducing the paper's Fig. 2 / Supplementary Table 1 comparison.

## Related tools

- **scMDC** (Method under benchmark test; multimodal deep clustering model run with CITE-Seq or SMAGE-seq defaults) — https://github.com/xianglin226/scMDC
- **BREM-SC** (Competing CITE-seq clustering method (hierarchical Bayesian mixture model, v0.2.0)) — https://github.com/tarot0410/BREMSC
- **CiteFuse** (Competing CITE-seq clustering method (similarity network fusion, v1.0.0)) — https://github.com/SydneyBioX/CiteFuse
- **Specter** (Competing method; landmark-based spectral clustering for single-cell and multimodal data) — https://github.com/canzarlab/Specter
- **Seurat V4** (Competing method; WNN procedure for multi-omics clustering (v4.0.4)) — https://github.com/satijalab/seurat
- **SCVIS** (Competing single-modal deep embedding method (v0.1.0)) — https://github.com/shahcompbio/scvis
- **IDEC** (Competing deep embedded clustering baseline) — https://github.com/XifengGuo/IDEC
- **SC3** (Competing single-modal spectral clustering method (v1.21.0)) — https://github.com/hemberg-lab/SC3
- **Tscan** (Competing single-modal method using PCA plus Gaussian mixture model clustering (v1.31.0)) — https://github.com/zji90/TSCAN
- **Cobolt** (Competing SMAGE-seq multimodal method (v1.0.0)) — https://github.com/epurdom/cobolt
- **scMM** (Competing SMAGE-seq multimodal deep generative method (v1.0.0)) — https://github.com/kodaim1115/scMM
- **Scanpy** (Preprocessing/normalization of mRNA and ADT data before benchmarking)
- **SKlearn** (Computation of AMI, NMI, and ARI external-validation metrics)

## Examples

```
from sklearn.metrics import adjusted_mutual_info_score, normalized_mutual_info_score, adjusted_rand_score; ami = adjusted_mutual_info_score(Y, pred)
```

## Evaluation signals

- Each results table must contain AMI, NMI, and ARI values for all methods on all datasets, plus per-metric ranks and paired t-test p-values
- scMDC should attain the top (or near-top) rank on each dataset, consistent with the paper's report that it 'consistently ranks number 1 across AMI, NMI, and ARI' on nine CITE-seq datasets
- One-sided paired t-tests of scMDC versus each competitor should yield statistically significant improvements across datasets
- All methods must be evaluated on the same Scanpy-preprocessed h5 inputs (X1, X2, Y, Batch) so scores are directly comparable
- For SMAGE-seq, scMDC should show the top mean rank, with Cobolt second-best for AMI and NMI

## Limitations

- All scMDC experiments require GPU hardware (Nvidia Tesla P100, 16 G), which may limit reproducibility on CPU-only systems
- The number of clusters K must be specified by the user after pretraining; no automatic estimation is provided (except when letting scMDC estimate K with n_clusters=-1 in the SMAGE-seq setting)
- No explicit limitations or caveats subsection is present in the provided text

## Evidence

- [results] ranks: [evidence span withheld — anchor status: not_found]
- [methods] t-tests: [evidence span withheld — anchor status: not_found]
- [discussion] scanpy: "The raw CITE-seq data is preprocessed and normalized by the Python package SCANPY48. mRNA and ADT data are normalized separately but using the same method."
- [methods] metrics: [evidence span withheld — anchor status: not_found]
- [discussion] gpu: "All experiments of scMDC in this study are conducted on Nvidia Tesla P100 (16 G) GPU."
