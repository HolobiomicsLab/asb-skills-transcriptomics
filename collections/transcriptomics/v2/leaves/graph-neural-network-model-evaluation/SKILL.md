---
name: graph-neural-network-model-evaluation
description: Use when benchmarking a graph neural network (e.g., DGMP vs. MutSigCV,
  EMOGI, deepDriver) on gene regulatory networks with multi-omics gene features, or
  when evaluating generalization to an independent driver-gene test set.
when_to_use_negative:
- The task requires patient-level predictions — DGMP averages SNVs, CNAs, DNA methylation,
  and gene expression across all samples, which ignores individual cancer patient
  characteristics.
- The input is a raw mutation count table without a directed gene-gene interaction
  network; DGMP requires a GRN for the DGCN graph convolution operation.
- The goal is to weight the DGCN and MLP embedding contributions differently — DGMP
  simply concatenates the three DGCN and one MLP embedding vectors without accounting
  for their differing contributions.
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_2640
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0602
tools:
- name: DGMP
  role: The DGCN+MLP model being trained, tested, and benchmarked on GRN datasets
  repo: https://github.com/NWPU-903PR/DGMP
- name: PyTorch 1.7.1
  role: Deep learning framework in which DGMP is developed and executed
- name: Anaconda
  role: Dependency configuration for the Python 3.7 DGMP environment
- name: torch-geometric 1.6.3
  role: Sparse graph handling library (with torch-sparse 0.6.8 and torch-scatter 2.0.6)
    required by DGMP
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.gpb.2022.11.004
    title: 'DGMP: Identifying Cancer Driver Genes by Jointing DGCN and MLP from Multi-Omics
      Genomic Data'
  source_content_hash: sha256:d34621ad076363d5f83fb54bfb5f1f8ab8bf4561856ce89c39e0d40d250b9099
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/NWPU-903PR/DGMP
  tools:
  - Anaconda
  - DGMP
  - PyTorch 1.7.1
  - Python
  - torch-geometric 1.6.3
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_2640
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# graph-neural-network-model-evaluation

## Summary

Reproducible setup and evaluation of DGMP, a directed graph convolutional network (DGCN) jointed with MLP for identifying cancer driver genes from multi-omics data, including independent testing on non-overlapping CancerMine-derived driver genes across DawnNet, KEGG, and RegNetwork GRNs.

## When to use

Use when benchmarking a graph neural network (e.g., DGMP vs. MutSigCV, EMOGI, deepDriver) on gene regulatory networks with multi-omics gene features, or when evaluating generalization to an independent driver-gene test set.

## When NOT to use

- The task requires patient-level predictions — DGMP averages SNVs, CNAs, DNA methylation, and gene expression across all samples, which ignores individual cancer patient characteristics.
- The input is a raw mutation count table without a directed gene-gene interaction network; DGMP requires a GRN for the DGCN graph convolution operation.
- The goal is to weight the DGCN and MLP embedding contributions differently — DGMP simply concatenates the three DGCN and one MLP embedding vectors without accounting for their differing contributions.

## Inputs

- Gene regulatory network dataset (DawnNet, KEGG, or RegNetwork) selected via --dataset
- Multi-omics gene features (SNVs, CNAs, DNA methylation, gene expression averaged across samples)
- Independent test set of CancerMine-derived driver genes non-overlapping with the training set

## Outputs

- 'logits' file in ./code/result/ with per-gene probabilities (column 1: non-cancer driver gene, column 2: cancer driver gene)
- 'acc' file with the mean 5-fold cross-validation accuracy
- AUPRC, AUROC, and TPR metrics for comparison against baseline methods

## How to apply

Configure a conda environment named DGMP with Python 3.7, then install PyTorch 1.7.1 with torchvision 0.8.2 and cudatoolkit 11.0 via conda, and torch-sparse 0.6.8, torch-scatter 2.0.6, and torch-geometric 1.6.3 via pip (required for sparse graph handling). Train and test with DGMP.py using the --gpu-no and --dataset parameters (set --gpu-no to -1 for CPU-only scenarios), selecting among the DawnNet, KEGG, or RegNetwork networks. Collect the two output variables from './code/result/': 'logits' (column 1 = non-cancer driver gene probability, column 2 = cancer driver gene probability) and 'acc' (mean 5-fold cross-validation accuracy). For generalization assessment, evaluate on an independent test set of CancerMine-derived driver genes that do not overlap the training set, comparing AUPRC, AUROC, and TPR against baselines.

## Related tools

- **DGMP** (The DGCN+MLP model being trained, tested, and benchmarked on GRN datasets) — https://github.com/NWPU-903PR/DGMP
- **PyTorch 1.7.1** (Deep learning framework in which DGMP is developed and executed)
- **Anaconda** (Dependency configuration for the Python 3.7 DGMP environment)
- **torch-geometric 1.6.3** (Sparse graph handling library (with torch-sparse 0.6.8 and torch-scatter 2.0.6) required by DGMP)

## Examples

```
cd code && python DGMP.py --gpu-no 0 --dataset DawnNet
```

## Evaluation signals

- Confirm './code/result/' contains both 'logits' and 'acc' after a run.
- Verify 'acc' holds the mean accuracy of the 5-fold cross-validation run on the selected dataset (e.g., DawnNet).
- Check logits has two columns: column 1 = non-cancer driver gene probability, column 2 = cancer driver gene probability.
- On the independent CancerMine test set, expect DGMP to achieve the best performance on DawnNet and RegNetwork (e.g., AUPRC 0.136/0.115, AUROC 0.706/0.757, TPR 0.769/0.864) versus baselines.
- Confirm the environment resolves: python 3.7, pytorch 1.7.1, torchvision 0.8.2, cudatoolkit 11.0, torch-sparse 0.6.8, torch-scatter 2.0.6, torch-geometric 1.6.3.

## Limitations

- Averaging omics across all samples ignores the specific characteristics of individual cancer patients.
- Simple concatenation of the three DGCN and one MLP embedding vectors does not account for their potentially differing contributions.
- On the independent CancerMine test, DGMP was best on DawnNet and RegNetwork only; performance on other networks (e.g., KEGG) may not lead.
- CUDA setup must be matched manually to the server environment (e.g., CUDA version 11.0); use --gpu-no -1 for CPU-only scenarios.

## Evidence

- [readme] conda install pytorch==1.7.1 torchvision==0.8.2 cudatoolkit=11.0: "Then, you need to install torch manually to fit in with your server environment (e.g. CUDA version 11.0). Run:"
- [methods] python DGMP.py --gpu-no 0 --dataset DawnNet: "python DGMP.py --gpu-no 0 --dataset DawnNet"
- [readme] The output of our model is supplied at './code/result/', which consist with two variates 'logits' and 'acc'.: "The output of our model is supplied at './code/result/', which consist with two variates 'logits' and 'acc'."
- [readme] the mean accuracy of 5-CV are in 'acc': "The probability of each gene is non-cancer driver gene (the first column) or cancer driver gene (the second column) have been stored in 'logits', and the mean accuracy of 5-CV are in 'acc'"
- [readme] torch-scatter and torch-sparse are required for dealing with sparse graph: "Besides, torch-scatter and torch-sparse are required for dealing with sparse graph, which can be installed using the following commands:"
- [results] In the independent test, DGMP achieves the best performance on the DawnNet and RegNetwork networks compared with all other methods (e.g., AUPRC 0.136/0.115, AUROC 0.706/0.757, TPR 0.769/0.864, respectively).: [evidence span withheld — anchor status: not_found]
