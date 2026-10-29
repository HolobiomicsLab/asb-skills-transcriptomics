---
name: directed-graph-convolution-on-gene-regulatory-networks
description: Use when reproducing cancer driver gene identification or ablation studies
  on a directed GRN (DawnNet, KEGG, RegNetwork) with multi-omics features, using the
  DGMP DGCN+MLP model.
when_to_use_negative:
- The graph network is undirected and directed regulation information is not part
  of the analysis goal (the DGCN specifically captures directed information)
- The task requires patient-level prediction rather than gene-level driver identification
  from sample-averaged 4-omics features
- You need to weight the four embedding vectors differently — DGMP simply concatenates
  three DGCN embeddings and one MLP embedding without weighting
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0611
- http://edamontology.org/topic_3473
tools:
- name: DGMP
  role: Main model implementing directed graph convolution (DGCN) jointed with MLP
    on multi-omics features over the directed GRN
  repo: https://github.com/NWPU-903PR/DGMP
- name: Anaconda
  role: Environment manager for creating the DGMP Python 3.7 dependency environment
  repo: https://www.anaconda.com/
- name: PyTorch 1.7.1
  role: Deep learning framework in which DGMP is developed and trained
- name: torch-geometric 1.6.3
  role: Graph neural network library used with torch-sparse and torch-scatter to handle
    the sparse directed gene graph
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.gpb.2022.11.004
    title: 'DGMP: Identifying Cancer Driver Genes by Jointing DGCN and MLP from Multi-Omics
      Genomic Data'
  source_content_hash: sha256:b4bebc9a6a7668f53d1d88a43a5ead3ea89434b69b4409804f524b0d48f9ef19
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/NWPU-903PR/DGMP
  tools:
  - Anaconda
  - DGMP
  - PyTorch 1.7.1
  - Python
  - torch-geometric
  - torch-geometric 1.6.3
  - torch-scatter
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0611
  - http://edamontology.org/topic_3473
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# directed-graph-convolution-on-gene-regulatory-networks

## Summary

Reproduce and ablate DGMP, a directed graph convolutional network (DGCN) jointed with an MLP, that learns multi-omics gene features and directed topological features of gene regulatory networks (GRNs) to identify cancer driver genes. This skill covers environment setup, per-variant training on DawnNet/KEGG/RegNetwork, and AUPRC/AUROC ablation validation.

## When to use

Use when reproducing cancer driver gene identification or ablation studies on a directed GRN (DawnNet, KEGG, RegNetwork) with multi-omics features, using the DGMP DGCN+MLP model.

## When NOT to use

- The graph network is undirected and directed regulation information is not part of the analysis goal (the DGCN specifically captures directed information)
- The task requires patient-level prediction rather than gene-level driver identification from sample-averaged 4-omics features
- You need to weight the four embedding vectors differently — DGMP simply concatenates three DGCN embeddings and one MLP embedding without weighting

## Inputs

- DawnNet multi-omics gene feature matrix (SNVs, CNAs, DNA methylation, gene expression averaged across samples)
- Directed gene regulatory network (GRN) supplied with the DGMP repository
- Ablation configuration toggles for DGCN/MLP/directionality components

## Outputs

- Per-gene cancer driver / non-cancer driver probability logits ('logits') in './code/result/'
- Mean accuracy of 5-fold cross-validation ('acc')
- Ablation table of AUPRC and AUROC (mean ± std) per variant, e.g. DGMP AUPRC 0.875 on DawnNet 5CV

## How to apply

Create an Anaconda environment (DGMP, Python 3.7) and install PyTorch 1.7.1, torchvision, cudatoolkit 11.0, plus torch-sparse 0.6.8 and torch-scatter 2.0.6 from the torch-1.7.1+cu110 PyTorch-Geometric wheels and torch-geometric 1.6.3. Load the DawnNet multi-omics gene feature matrix and the directed gene regulatory network from the DGMP repository. Train each ablated variant (DGCN alone, MLP alone, DGCN+MLP, toggling directionality) by running DGMP.py with the --dataset DawnNet flag and --gpu-no 0 (use -1 for CPU). Evaluate each variant with 5-fold cross-validation, collecting the per-gene driver/non-driver probability logits and mean CV accuracy written to './code/result/', then compute AUPRC and AUROC aggregated as mean ± std across folds.

## Related tools

- **DGMP** (Main model implementing directed graph convolution (DGCN) jointed with MLP on multi-omics features over the directed GRN) — https://github.com/NWPU-903PR/DGMP
- **Anaconda** (Environment manager for creating the DGMP Python 3.7 dependency environment) — https://www.anaconda.com/
- **PyTorch 1.7.1** (Deep learning framework in which DGMP is developed and trained)
- **torch-geometric 1.6.3** (Graph neural network library used with torch-sparse and torch-scatter to handle the sparse directed gene graph)

## Examples

```
cd code && python DGMP.py --gpu-no 0 --dataset DawnNet
```

## Evaluation signals

- AUPRC of DGMP (DGCN + MLP) on DawnNet 5CV should be 0.875, 0.047 higher than DGCN alone and 0.074 higher than MLP alone
- Per-variant AUPRC/AUROC mean ± std across 5 folds must match the Table 3 ablation results
- './code/result/' must contain the two output variables: per-gene probability 'logits' and the 5-CV mean accuracy 'acc'
- Jointing MLP with DGCN should offset the performance degradation of DGCN alone, confirming all components contribute
- DGMP should outperform state-of-the-art baselines (e.g. EMOGI, deepDriver) on all three GRNs: DawnNet, KEGG, and RegNetwork

## Limitations

- Averaging SNVs, CNAs, DNA methylation, and gene expression across all samples ignores specific characteristics of individual cancer patients
- DGMP simply concatenates the three DGCN embedding vectors and the one MLP embedding vector, not accounting for their potentially different contributions
- On CPU-only machines the --gpu-no parameter must be set to -1, which may affect runtime

## Evidence

- [abstract] The ablation experimental results on the DawnNet network indicate that introducing MLP into DGCN can offset the performance degradation of DGCN: "The ablation experimental results on the DawnNet network indicate that introducing MLP into DGCN can offset the performance degradation of DGCN"
- [methods] Training command: "python DGMP.py --gpu-no 0 --dataset DawnNet"
- [methods] Environment: "Our project is developed using PyTorch 1.7.1 in Python 3.7."
- [discussion] 4-omics averaging: "DGMP averages the SNVs, CNAs, DNA methylation, and gene expression of all samples to obtain 4-omics features to represent the genes for each cancer"
- [readme] Result output variables: "The output of our model is supplied at './code/result/', which consist with two variates 'logits' and 'acc'."
