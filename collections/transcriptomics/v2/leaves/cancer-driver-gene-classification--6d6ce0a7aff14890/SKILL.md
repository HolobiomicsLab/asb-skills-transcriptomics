---
name: cancer-driver-gene-classification
description: Use when genes must be classified cancer driver vs non-driver from multi-omics
  features averaged per gene over a directed GRN such as DawnNet, KEGG, or RegNetwork,
  e.g. to reproduce the DGMP 5-fold cross-validation benchmark.
when_to_use_negative:
- The input graph is an undirected PPI network rather than a directed GRN — DGMP's
  DGCN captures directed regulation information (forward/backward, Zin/Zout) that
  undirected graphs cannot provide.
- Per-patient or per-sample predictions are required — DGMP averages SNVs, CNAs, DNA
  methylation, and gene expression across all samples, which the authors note ignores
  the specific characteristics of individual cancer patients.
- The goal is recurrent-mutation significance testing (MutSigCV-style) rather than
  supervised classification from integrated multi-omics features.
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0199
- http://edamontology.org/topic_0659
tools:
- name: DGMP
  role: The joint DGCN+MLP classifier trained and tested via DGMP.py, producing per-gene
    driver probability logits and mean 5-CV accuracy
  repo: https://github.com/NWPU-903PR/DGMP
- name: PyTorch 1.7.1
  role: Deep-learning framework in which the DGCN and MLP components are implemented
    and trained (with torchvision 0.8.2 / cudatoolkit 11.0)
- name: torch-geometric 1.6.3
  role: Graph-learning stack (with torch-sparse 0.6.8 and torch-scatter 2.0.6) providing
    the sparse graph operations needed for the directed GRN convolution
- name: Anaconda
  role: Environment manager used to create the DGMP conda environment pinning Python
    3.7, PyTorch 1.7.1, torchvision, and cudatoolkit 11.0
- name: DiGC
  role: Upstream directed graph convolutional network (DiGCN) implementation whose
    template DGMP borrows for its DGCN component
  repo: https://github.com/flyingtango/DiGC
provenance:
  source_task_ids:
  - task_001
  - task_003
  - task_005
  - task_002
  source_papers:
  - doi: 10.1016/j.gpb.2022.11.004
    title: 'DGMP: Identifying Cancer Driver Genes by Jointing DGCN and MLP from Multi-Omics
      Genomic Data'
  source_content_hash: sha256:8dd6296e9a52a6f52ff359f26ddc0e56e9be3ad8a34aaf0ced9923ffe7384f9e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/NWPU-903PR/DGMP
  tools:
  - Anaconda
  - DGMP
  - DiGC
  - PyTorch 1.7.1
  - Python
  - torch-geometric
  - torch-geometric 1.6.3
  - torch-scatter
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0199
  - http://edamontology.org/topic_0659
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5625
n_claims: 16
---

# cancer-driver-gene-classification

## Summary

Trains DGMP — a directed graph convolutional network (DGCN) jointed with an MLP, whose three DGCN embeddings (ZF, Zin, Zout) and MLP embedding (ZMLP) are concatenated into a fully connected network — on per-gene 4-omics features (SNVs, CNAs, DNA methylation, gene expression) over directed GRNs (DawnNet, KEGG, RegNetwork) to output per-gene cancer-driver probabilities under 5-fold cross-validation.

## When to use

Use when genes must be classified cancer driver vs non-driver from multi-omics features averaged per gene over a directed GRN such as DawnNet, KEGG, or RegNetwork, e.g. to reproduce the DGMP 5-fold cross-validation benchmark.

## When NOT to use

- The input graph is an undirected PPI network rather than a directed GRN — DGMP's DGCN captures directed regulation information (forward/backward, Zin/Zout) that undirected graphs cannot provide.
- Per-patient or per-sample predictions are required — DGMP averages SNVs, CNAs, DNA methylation, and gene expression across all samples, which the authors note ignores the specific characteristics of individual cancer patients.
- The goal is recurrent-mutation significance testing (MutSigCV-style) rather than supervised classification from integrated multi-omics features.

## Inputs

- Directed GRN dataset shipped with the DGMP repository (DawnNet, KEGG, or RegNetwork), selected via the --dataset flag
- Per-gene 4-omics feature matrix (SNVs, CNAs, DNA methylation, gene expression averaged across all samples)
- Binary cancer driver / non-cancer driver gene labels for 5-fold cross-validation

## Outputs

- 'logits' in ./code/result/: two-column per-gene probability matrix (column 1: non-cancer driver gene, column 2: cancer driver gene)
- 'acc' in ./code/result/: mean accuracy over the 5 cross-validation folds
- AUPRC/AUROC values scored from logits against held-out labels (e.g. DawnNet AUPRC 0.875, AUROC 0.889 in 5-CV)

## How to apply

Create an Anaconda environment named DGMP (Python 3.7), install PyTorch 1.7.1 with torchvision 0.8.2 and cudatoolkit 11.0 via conda, then pip-install torch-sparse 0.6.8, torch-scatter 2.0.6, and torch-geometric 1.6.3 from the torch-1.7.1+cu110 wheels for sparse graph handling. Load the directed GRN dataset shipped with the repository (selected via --dataset, e.g. DawnNet) together with its multi-omics gene features — SNVs, CNAs, DNA methylation, and gene expression averaged across all samples. Run training and 5-fold cross-validation with 'python DGMP.py --gpu-no 0 --dataset DawnNet' (set --gpu-no to -1 for CPU-only execution); the model concatenates the three DGCN embeddings (ZF, Zin, Zout) and one MLP embedding (ZMLP) into a fully connected neural network that scores every gene. Read 'logits' (two columns: non-cancer driver vs cancer driver probability per gene) and 'acc' (mean 5-CV accuracy) from './code/result/', then compute AUPRC/AUROC against the fold labels. On DawnNet the expected benchmark is AUPRC 0.875 and AUROC 0.889, exceeding EMOGI, NRFD, DeepWalk+SVM, PageRank, HotNet2, and MutSigCV.

## Related tools

- **DGMP** (The joint DGCN+MLP classifier trained and tested via DGMP.py, producing per-gene driver probability logits and mean 5-CV accuracy) — https://github.com/NWPU-903PR/DGMP
- **PyTorch 1.7.1** (Deep-learning framework in which the DGCN and MLP components are implemented and trained (with torchvision 0.8.2 / cudatoolkit 11.0))
- **torch-geometric 1.6.3** (Graph-learning stack (with torch-sparse 0.6.8 and torch-scatter 2.0.6) providing the sparse graph operations needed for the directed GRN convolution)
- **Anaconda** (Environment manager used to create the DGMP conda environment pinning Python 3.7, PyTorch 1.7.1, torchvision, and cudatoolkit 11.0)
- **DiGC** (Upstream directed graph convolutional network (DiGCN) implementation whose template DGMP borrows for its DGCN component) — https://github.com/flyingtango/DiGC

## Examples

```
cd code; python DGMP.py --gpu-no 0 --dataset DawnNet
```

## Evaluation signals

- After a run, './code/result/' contains both 'logits' and 'acc'.
- 'logits' holds exactly two probability columns per gene — column 1 non-cancer driver, column 2 cancer driver — with one row for every gene in the GRN.
- 'acc' stores a single mean accuracy aggregated over the 5 cross-validation folds.
- AUPRC/AUROC recomputed from logits on DawnNet 5-CV should approximate the reported 0.875/0.889, exceeding the six baselines by 0.054–0.355 (AUPRC) and 0.053–0.284 (AUROC).
- Ablation sanity check: DGCN-only and MLP-only variants should score ~0.047 and ~0.074 lower AUPRC than the full DGCN+MLP model on DawnNet.

## Limitations

- Averaging SNVs, CNAs, DNA methylation, and gene expression across all samples ignores the specific characteristics of individual cancer patients (stated limitation).
- The three DGCN embeddings and one MLP embedding are simply concatenated, so their differing contributions are not weighted (stated limitation).
- Results are tied to the pinned stack (Python 3.7, PyTorch 1.7.1, torch-geometric 1.6.3, CUDA 11.0); CPU-only runs must set --gpu-no to -1.
- Generalization to the non-overlapping CancerMine-derived driver test set is much weaker than 5-CV (AUPRC 0.136 on DawnNet vs 0.875 within cross-validation).

## Evidence

- [abstract] we propose a novel method (called DGMP) to identify cancer driver genes by jointing directed graph convolutional network (DGCN) and multilayer perceptron (MLP): "we propose a novel method (called DGMP) to identify cancer driver genes by jointing directed graph convolutional network (DGCN) and multilayer perceptron (MLP)"
- [discussion] DGMP averages the SNVs, CNAs, DNA methylation, and gene expression of all samples to obtain 4-omics features to represent the genes for each cancer: "DGMP averages the SNVs, CNAs, DNA methylation, and gene expression of all samples to obtain 4-omics features to represent the genes for each cancer"
- [discussion] Three embedding feature vectors from DGCN and one embedding feature vector from MLP are concatenated to feed into a fully connected neural network: "Three embedding feature vectors from DGCN and one embedding feature vector from MLP are concatenated to feed into a fully connected neural network"
- [methods] Training and testing our DGMP model with the following commands: "Training and testing our DGMP model with the following commands"
- [readme] The probability of each gene is non-cancer driver gene (the first column) or cancer driver gene (the second column) have been stored in 'logits': "The probability of each gene is non-cancer driver gene (the first column) or cancer driver gene (the second column) have been stored in 'logits'"
- [readme] torch-scatter and torch-sparse are required for dealing with sparse graph: "torch-scatter and torch-sparse are required for dealing with sparse graph"
