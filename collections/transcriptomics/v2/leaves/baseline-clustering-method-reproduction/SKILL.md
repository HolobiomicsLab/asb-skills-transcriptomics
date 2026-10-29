---
name: baseline-clustering-method-reproduction
description: Use when benchmarking a new scRNA-seq clustering method against published
  baselines on the same down-sampled cell sets, or when verifying that scDCC with
  pairwise constraints outperforms competing methods on the four small 2100-cell datasets.
when_to_use_negative:
- The datasets are not the 2100-cell down-sampled samples (e.g. the full 10X CITE
  PBMC or human liver datasets) — use the dataset-specific wrappers scDCC_pairwise_CITE_PBMC.py
  or scDCC_pairwise_Human_liver.py instead
- The goal is to estimate the number of clusters rather than compare benchmark performance
  — use the scDCC_estimating_number_of_clusters pipeline instead
- You only need scDCC's own constrained clustering on new data with prior knowledge,
  without reproducing the multi-baseline benchmark
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scDCC
  role: Method under test; run both unsupervised (0 constraints) and with soft pairwise
    constraints
  repo: https://github.com/ttgump/scDCC
- name: CIDR
  role: Competing clustering baseline run on the same 2100-cell samples
  repo: VCCRI/CIDR
- name: DCA
  role: Denoising baseline combined with k-means in the comparison
  repo: theislab/dca
- name: DEC-keras
  role: Deep embedding clustering baseline in the comparison
  repo: XifengGuo/DEC-keras
- name: MPSSC
  role: Spectral clustering baseline (multiple doubly stochastic affinity matrices)
  repo: ishspsy/project
- name: scvi-tools
  role: Source of SCVI used as SCVI+k-means baseline
  repo: scverse/scvi-tools
- name: scvis
  role: Source of scvis used as SCVIS+k-means baseline
  repo: shahcompbio/scvis
- name: PyTorch
  role: Framework (1.5.1+cu101) used to run scDCC.py
- name: Scanpy
  role: Preprocessing library (1.0.4) used in the scDCC environment
- name: Nvidia Tesla P100
  role: GPU hardware on which the benchmark was executed
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-021-22008-3
    title: Model-based deep embedding for constrained clustering analysis of single
      cell RNA-seq data
  source_content_hash: sha256:2bd8eb5869439439708d24a40b2475a20643b4260e6bfcb0a4deb42fe3cd495e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ttgump/scDCC
  tools:
  - CIDR
  - DCA
  - DEC
  - DEC-keras
  - MPSSC
  - Nvidia Tesla P100
  - PyTorch
  - Python
  - Scanpy
  - scDCC
  - scVI
  - scvi-tools
  - scvis
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 15
---

# baseline-clustering-method-reproduction

## Summary

Reproduces the scDCC paper's head-to-head comparison of scDCC (with 0 constraints and with pairwise constraints) against twelve competing scRNA-seq clustering baselines (CIDR, DCA+k-means, DEC, MPSSC, PCA+k-means, SCVI+k-means, SCVIS+k-means, SIMLR, SC3, Seurat, COP K-means, MPC K-means) on 2100-cell down-sampled datasets, evaluating with NMI, CA, and ARI.

## When to use

Use when benchmarking a new scRNA-seq clustering method against published baselines on the same down-sampled cell sets, or when verifying that scDCC with pairwise constraints outperforms competing methods on the four small 2100-cell datasets.

## When NOT to use

- The datasets are not the 2100-cell down-sampled samples (e.g. the full 10X CITE PBMC or human liver datasets) — use the dataset-specific wrappers scDCC_pairwise_CITE_PBMC.py or scDCC_pairwise_Human_liver.py instead
- The goal is to estimate the number of clusters rather than compare benchmark performance — use the scDCC_estimating_number_of_clusters pipeline instead
- You only need scDCC's own constrained clustering on new data with prior knowledge, without reproducing the multi-baseline benchmark

## Inputs

- Four small scRNA-seq 2100-cell down-sampled datasets from https://figshare.com/articles/dataset/scDCC_data/21563517
- Ground-truth cell labels for each 2100-cell sample
- Reference implementations of the competing baselines (CIDR, DCA, DEC-keras, MPSSC, scvi-tools, scvis, SC3, Seurat)

## Outputs

- Per-method NMI, CA, and ARI values for scDCC (0 constraints) and scDCC (pairwise constraints) on each dataset
- Per-method NMI, CA, and ARI values for the twelve competing baselines
- Combined comparison table saved as benchmark_comparison_table.csv

## How to apply

Download the four small 2100-cell scRNA-seq samples from the figshare deposit (https://figshare.com/articles/dataset/scDCC_data/21563517). Run scDCC in unsupervised mode (0 constraints, equivalent to scDeepCluster) using scDCC.py under Python 3.6.8 with PyTorch 1.5.1+cu101 and Scanpy 1.0.4 on an Nvidia Tesla P100, then run scDCC with generated soft pairwise constraints via --n_pairwise plus --gamma (clustering loss weight), --ml_weight (must-link), and --cl_weight (cannot-link). Next, run each of the twelve competing baselines — CIDR, DCA+k-means, DEC, MPSSC, PCA+k-means, SCVI+k-means, SCVIS+k-means, SIMLR, SC3, Seurat, COP K-means, MPC K-means — using their reference implementations (e.g. VCCRI/CIDR in R, theislab/dca, XifengGuo/DEC-keras, ishspsy/project for MPSSC, scverse/scvi-tools, shahcompbio/scvis) on the identical 2100-cell samples. For every method compute NMI, CA, and ARI against the ground-truth cell labels on each dataset, assemble the per-method metric values into a single table (benchmark_comparison_table.csv), and check that scDCC with pairwise constraints attains higher NMI, CA, and ARI than all competing methods, consistent with the paper's reported result.

## Related tools

- **scDCC** (Method under test; run both unsupervised (0 constraints) and with soft pairwise constraints) — https://github.com/ttgump/scDCC
- **CIDR** (Competing clustering baseline run on the same 2100-cell samples) — VCCRI/CIDR
- **DCA** (Denoising baseline combined with k-means in the comparison) — theislab/dca
- **DEC-keras** (Deep embedding clustering baseline in the comparison) — XifengGuo/DEC-keras
- **MPSSC** (Spectral clustering baseline (multiple doubly stochastic affinity matrices)) — ishspsy/project
- **scvi-tools** (Source of SCVI used as SCVI+k-means baseline) — scverse/scvi-tools
- **scvis** (Source of scvis used as SCVIS+k-means baseline) — shahcompbio/scvis
- **PyTorch** (Framework (1.5.1+cu101) used to run scDCC.py)
- **Scanpy** (Preprocessing library (1.0.4) used in the scDCC environment)
- **Nvidia Tesla P100** (GPU hardware on which the benchmark was executed)

## Examples

```
python scDCC.py --n_pairwise 100 --gamma 1 --ml_weight 1 --cl_weight 1
```

## Evaluation signals

- benchmark_comparison_table.csv contains rows for all fourteen method variants (scDCC 0-constraint, scDCC pairwise, and twelve baselines) across all four datasets
- Every cell in the table has a numeric NMI, CA, and ARI value computed against the same ground-truth cell labels
- scDCC with pairwise constraints attains strictly higher NMI, CA, and ARI than every competing method on the four small datasets
- scDCC with 0 constraints (scDeepCluster equivalent) outperforms most, though not necessarily all, competing methods
- Runs reproduce on the stated environment: Python 3.6.8, PyTorch 1.5.1+cu101, Scanpy 1.0.4, Nvidia Tesla P100

## Limitations

- Baselines without GPU/deep-learning requirements (CIDR, SC3, Seurat) and with them (DEC, DCA, scvis, scvi-tools) have heterogeneous reference implementations; each must use its own reference code base
- The comparison is restricted to 2100-cell down-sampled samples, so conclusions do not automatically extend to full-size datasets of tens of thousands of cells
- Constrained scDCC depends on generated soft pairwise constraints; its advantage requires a sensible --n_pairwise count and tuned --gamma, --ml_weight, --cl_weight weights

## Evidence

- [results] On the randomly selected 2100-cell samples from each of the four datasets, scDCC with 0 constraints outperformed most competing scRNA-seq clustering methods: [evidence span withheld — anchor status: not_found]
- [results] incorporating prior information as constraints: [evidence span withheld — anchor status: not_found]
- [intro] soft pairwise constraints added to loss: "we convert (partial) prior knowledge into soft pairwise constraints and add them as additional terms into the loss"
- [methods] figshare dataset deposit: "Datasets used in the study is available in: https://figshare.com/articles/dataset/scDCC_data/21563517"
- [methods] environment versions: [evidence span withheld — anchor status: not_found]
- [methods] constraint generation flags: "--n_pairwise: number of pairwise constraints want to generate"
- [readme] scDCC README framing: "Most existing methods support unsupervised clustering without the a priori exploitation of any domain knowledge. When confronted by the high dimensionality and pervasive dropout events of scRNA-Seq"
