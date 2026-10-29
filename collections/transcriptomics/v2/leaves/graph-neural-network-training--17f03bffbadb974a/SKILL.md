---
name: graph-neural-network-training
description: Use when training a GNN on a directed gene regulatory network with per-gene
  multi-omics features (SNVs, CNAs, DNA methylation, gene expression) to predict binary
  gene labels such as driver/non-driver, or when reproducing ablation studies of DGCN/MLP
  components.
when_to_use_negative:
- The task requires patient-level predictions — DGMP averages omics across all samples,
  ignoring individual patient-specific characteristics
- The graph is undirected and topology-free gene features suffice — a plain MLP or
  non-graph classifier is more appropriate than DGCN
- Embedding vectors have genuinely different contributions requiring learned weighting
  — DGMP simply concatenates the three DGCN and one MLP vectors without weighting
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_2640
- http://edamontology.org/topic_0602
tools:
- name: DGMP
  role: Main model and training script (DGMP.py) that joints DGCN and MLP for driver
    gene classification on directed GRNs
  repo: https://github.com/NWPU-903PR/DGMP
- name: PyTorch 1.7.1
  role: Deep learning framework in which DGMP is developed and trained
- name: torch-geometric 1.6.3
  role: Graph neural network library used with torch-scatter and torch-sparse to handle
    the sparse directed graph
  repo: https://github.com/flyingtango/DiGC
- name: Anaconda
  role: Environment manager for creating the DGMP python=3.7 environment and installing
    pinned dependencies
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.gpb.2022.11.004
    title: 'DGMP: Identifying Cancer Driver Genes by Jointing DGCN and MLP from Multi-Omics
      Genomic Data'
  source_content_hash: sha256:f312514351a3f272c81fa587d562f8176555ee00c7e09aa315503468a9b6ded6
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
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_2640
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# graph-neural-network-training

## Summary

Train and evaluate graph neural network models for gene-level classification by combining a directed graph convolutional network (DGCN) operating on a directed gene regulatory network with an MLP branch over multi-omics gene features, as implemented in DGMP for cancer driver gene identification on networks like DawnNet, KEGG, and RegNetwork.

## When to use

Use when training a GNN on a directed gene regulatory network with per-gene multi-omics features (SNVs, CNAs, DNA methylation, gene expression) to predict binary gene labels such as driver/non-driver, or when reproducing ablation studies of DGCN/MLP components.

## When NOT to use

- The task requires patient-level predictions — DGMP averages omics across all samples, ignoring individual patient-specific characteristics
- The graph is undirected and topology-free gene features suffice — a plain MLP or non-graph classifier is more appropriate than DGCN
- Embedding vectors have genuinely different contributions requiring learned weighting — DGMP simply concatenates the three DGCN and one MLP vectors without weighting

## Inputs

- Directed gene regulatory network (e.g., DawnNet, KEGG, or RegNetwork)
- Multi-omics gene feature matrix (averaged SNVs, CNAs, DNA methylation, gene expression per cancer type)
- Gene driver/non-driver labels for supervised training
- Command-line flags: --gpu-no (0 for GPU, -1 for CPU), --dataset (network name)

## Outputs

- Per-gene probability logits ('logits': column 1 = non-driver, column 2 = driver) written to './code/result/'
- Mean 5-fold cross-validation accuracy ('acc')
- AUPRC and AUROC values (mean ± std across folds) for full model and ablated variants

## How to apply

Create an Anaconda environment (python=3.7) and install PyTorch 1.7.1, torchvision 0.8.2, cudatoolkit 11.0, plus torch-sparse 0.6.8, torch-scatter 2.0.6, and torch-geometric 1.6.3 from the PyTorch-Geometric wheels for torch-1.7.1+cu110. Load the multi-omics gene feature matrix and the directed GRN (e.g., DawnNet) supplied with the DGMP repository; DGCN captures directed regulation information over second-order neighbors while the MLP branch weighs gene features to mitigate topological bias. Train via DGMP.py with --gpu-no 0 (use -1 for CPU-only) and --dataset DawnNet; three DGCN embedding vectors and one MLP embedding vector are concatenated into a fully connected neural network for classification. Evaluate with 5-fold cross-validation, reading per-gene driver/non-driver probability 'logits' and mean CV 'acc' from './code/result/'. For ablations, toggle DGCN/MLP/directionality components and compute AUPRC/AUROC per variant (e.g., DGMP AUPRC 0.875 vs 0.828 DGCN-only and 0.801 MLP-only on DawnNet 5CV).

## Related tools

- **DGMP** (Main model and training script (DGMP.py) that joints DGCN and MLP for driver gene classification on directed GRNs) — https://github.com/NWPU-903PR/DGMP
- **PyTorch 1.7.1** (Deep learning framework in which DGMP is developed and trained)
- **torch-geometric 1.6.3** (Graph neural network library used with torch-scatter and torch-sparse to handle the sparse directed graph) — https://github.com/flyingtango/DiGC
- **Anaconda** (Environment manager for creating the DGMP python=3.7 environment and installing pinned dependencies)

## Examples

```
cd code && python DGMP.py --gpu-no 0 --dataset DawnNet
```

## Evaluation signals

- On DawnNet in 5CV, full DGMP (DGCN + MLP) AUPRC reaches 0.875, exceeding DGCN alone by 0.047 and MLP alone by 0.074 — reproduce this ordering and magnitudes in Table 3 ablation results
- Output directory './code/result/' must contain two files, 'logits' (two-column per-gene probabilities) and 'acc' (mean 5-CV accuracy)
- Per-variant AUPRC/AUROC mean ± std across folds must match the published Table 3 ablation values
- Introducing MLP into DGCN must offset DGCN's performance degradation rather than degrade it
- DGMP should outperform state-of-the-art baselines (e.g., EMOGI, deepDriver) on all three networks: DawnNet, KEGG, and RegNetwork

## Limitations

- Averaging SNVs, CNAs, DNA methylation, and gene expression across all samples ignores specific characteristics of individual cancer patients
- Simple concatenation of the three DGCN and one MLP embedding vectors does not account for their possibly different contributions
- DGCN alone shows degraded performance relative to the joint model, so the MLP branch is required for best results

## Evidence

- [methods] Training via DGMP.py with dataset flag: "python DGMP.py --gpu-no 0 --dataset DawnNet"
- [readme] Training and testing commands with gpu-no and dataset parameters: [evidence span withheld — anchor status: not_found]
- [results] Ablation result on DawnNet: [evidence span withheld — anchor status: not_found]
- [readme] Output artifacts of training: "The output of our model is supplied at './code/result/', which consist with two variates 'logits' and 'acc'."
- [readme] Pinned dependency installation: "pip install torch-sparse==0.6.8 -f https://pytorch-geometric.com/whl/torch-1.7.1+cu110.html"
