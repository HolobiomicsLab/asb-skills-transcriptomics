---
name: atlas-query-label-prediction-benchmarking
description: Use when two methods for TCR/transcriptome embedding or antigen specificity
  prediction must be compared on identical preprocessed atlas-query splits, and per-dataset
  F1-Score/NMI gains with p-values are the required deliverable.
when_to_use_negative:
- When only a single method is being evaluated without a competing baseline — no benchmark
  comparison table can be formed
- When query datasets cannot be mapped to a shared reference atlas (no common label
  space or compatible embedding for reference mapping)
- When TCR input modalities cannot be matched between methods (e.g., full multimodal
  input vs. CDR3β-only), making the comparison unfair
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: mvTCR
  role: Multimodal embedding model trained with CDR3β-only TCR input for comparability;
    provides query-to-atlas reference mapping and label prediction/clustering of query
    cells
  repo: https://github.com/SchubertLab/mvTCR
- name: tessa
  role: Baseline Bayesian TCR functional landscape estimation model supervised with
    scRNA-Seq, run on identically preprocessed datasets for the benchmark
- name: scArches
  role: Reference-mapping approach combined with mvTCR to map new multimodal studies
    into reference atlases
- name: Scanpy
  role: Hosts mvTCR's shared embedding in standard single-cell workflows used for
    clustering and evaluation
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-024-49806-9
    title: Multi-modal generative modeling for joint analysis of single-cell T cell
      receptor and gene expression data
  source_content_hash: sha256:7ff4af0efdcc566197bd541c703edfa8f9de08d1163d6c774609687460117f85
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SchubertLab/mvTCR
  tools:
  - Scanpy
  - mvTCR
  - scArches
  - tessa
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# atlas-query-label-prediction-benchmarking

## Summary

Benchmark competing multimodal embedding methods (e.g., mvTCR vs. tessa) by mapping query T cell datasets onto a reference atlas, predicting query labels, and comparing F1-Score and NMI across datasets with paired significance tests.

## When to use

Use when two methods for TCR/transcriptome embedding or antigen specificity prediction must be compared on identical preprocessed atlas-query splits, and per-dataset F1-Score/NMI gains with p-values are the required deliverable.

## When NOT to use

- When only a single method is being evaluated without a competing baseline — no benchmark comparison table can be formed
- When query datasets cannot be mapped to a shared reference atlas (no common label space or compatible embedding for reference mapping)
- When TCR input modalities cannot be matched between methods (e.g., full multimodal input vs. CDR3β-only), making the comparison unfair

## Inputs

- Preprocessed 10x single-cell T cell dataset (>60k cells) with gene expression and paired TCR sequences
- Minervina T cell dataset with antigen specificity annotations
- Cell type, clonotype, and specificity annotations added during preprocessing
- Trained reference models (e.g., mvTCR v0.1.3 checkpoint) and held-out query datasets

## Outputs

- Per-dataset F1-Score and NMI for each method
- Average percent gains of one method over the other (e.g., 23.6% F1-Score, 27.7% NMI for mvTCR over tessa)
- Paired-test p-values (e.g., p=4.7e-6 for F1-Score, p=4.6e-6 for NMI)
- benchmark_comparison.csv comparison table

## How to apply

Preprocess the reference atlas and query datasets identically (e.g., the 10x >60k-cell dataset and Minervina T cell data using the mvTCR preprocessing notebook tutorials/01_preprocessing.ipynb, adding cell type, clonotype, and specificity annotations). Train each method on the atlas with matched input modality — for a fair mvTCR-vs-tessa comparison, restrict mvTCR's TCR input to the CDR3β sequence only so representations are comparable to tessa's clonotype-level Bayesian functional landscape. Map query data to the reference (mvTCR supports scArches-style reference mapping) and predict query cell labels per dataset. Compute F1-Score for label prediction and NMI for clustering per dataset, aggregate into average percent gains for one method over the other, and run paired significance tests across datasets (expected for mvTCR vs. tessa: ~23.6% F1 gain, p=4.7e-6; ~27.7% NMI gain, p=4.6e-6). Write per-dataset scores, average gains, and p-values to a comparison table (benchmark_comparison.csv). Use mvTCR v0.1.3 with PyTorch v1.8.0 to reproduce the published settings; trained models are available on Zenodo (doi:10.5281/zenodo.7215447).

## Related tools

- **mvTCR** (Multimodal embedding model trained with CDR3β-only TCR input for comparability; provides query-to-atlas reference mapping and label prediction/clustering of query cells) — https://github.com/SchubertLab/mvTCR
- **tessa** (Baseline Bayesian TCR functional landscape estimation model supervised with scRNA-Seq, run on identically preprocessed datasets for the benchmark)
- **scArches** (Reference-mapping approach combined with mvTCR to map new multimodal studies into reference atlases)
- **Scanpy** (Hosts mvTCR's shared embedding in standard single-cell workflows used for clustering and evaluation)

## Examples

```
conda create --name mvTCR python=3.8.8 -y && conda activate mvTCR && pip install mvtcr==0.1.3 && conda install nb_conda_kernels -y
```

## Evaluation signals

- Per-dataset F1-Score and NMI are computed for both methods on identical atlas-query splits; mvTCR CDR3β-only should exceed tessa on all datasets
- Average improvement should approximate 23.6% F1-Score (p-value 4.7e-6) and 27.7% NMI (p-value 4.6e-6) with paired significance tests
- benchmark_comparison.csv contains per-dataset scores, average gains, and p-values with no missing entries
- Reproduced runs use mvTCR v0.1.3 with PyTorch v1.8.0, matching the reproducibility repo setup (Python 3.8.8 conda environment)
- Preprocessing follows tutorials/01_preprocessing.ipynb with required cell type, clonotype, and specificity annotations present

## Limitations

- Re-run models may diverge from published models due to differences in compute clusters, hardware, and CUDA versions; pre-trained models should be downloaded from Zenodo (doi:10.5281/zenodo.7215447)
- Hyperparameter optimization is expensive (48 GPU-hours per experiment in the paper's setup)
- The TCR representation uses only the CDR3 sequence; VDJ-gene encoding and pre-trained TCR embedding models are not incorporated, which bounds the achievable prediction performance
- UMAP visualizations in downstream figures may not be fully recreated due to reproducibility issues in the underlying library

## Evidence

- [results] mvTCR trained with CDR3β-only input outperformed tessa on atlas-query mapping across all datasets, with an average improvement of 23.6% in F1-Score (p-value: 4.7e-6) for prediction and 27.7% in NMI (p-value: 4.6e-6) for clustering.: [evidence span withheld — anchor status: not_found]
- [discussion] When combined with tools like scArches, mvTCR adeptly mapped new multimodal studies into reference atlases.: "When combined with tools like scArches, mvTCR adeptly mapped new multimodal studies into reference atlases"
- [readme] The experiments were run with mvTCR v0.1.3 together with PyTorch v1.8.0.: "The experiments were run with mvTCR v0.1.3 together with PyTorch v1.8.0."
- [methods] tutorials/01_preprocessing.ipynb : adding requiered annotation and filtering on the data.: "tutorials/01_preprocessing.ipynb : adding requiered annotation and filtering on the data."
- [discussion] mvTCR's embedding capability surpasses that of unimodal representations and other multimodal methods, as demonstrated through its efficacy in antigen and avidity prediction, reference mapping: "mvTCR’s embedding capability surpasses that of unimodal representations and other multimodal methods, as demonstrated through its efficacy in antigen and avidity prediction, reference mapping"
