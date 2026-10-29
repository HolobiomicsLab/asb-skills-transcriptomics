---
name: multi-omics-feature-learning
description: Use when labeled driver/non-driver genes are available on a directed
  GRN (DawnNet, KEGG, RegNetwork) with per-gene multi-omics features, and the goal
  is to predict cancer driver genes via graph + feature learning.
when_to_use_negative:
- The network is an undirected PPI network rather than a directed GRN — DGCN is designed
  to capture directed regulation information
- Only single-gene features without graph topology are available and a purely feature-based
  classifier is intended; DGMP's design couples multi-omics features with GRN topological
  structure
- Patient-level prediction is required — DGMP averages omics across all samples, which
  ignores the specific characteristics of individual cancer patients
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_2640
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0749
tools:
- name: DGMP
  role: 'Core model: jointing DGCN and MLP to learn multi-omics gene features and
    GRN topological features for cancer driver gene identification'
  repo: https://github.com/NWPU-903PR/DGMP
- name: PyTorch 1.7.1
  role: Deep learning framework in which DGMP is developed and run
  repo: https://github.com/NWPU-903PR/DGMP
- name: torch-geometric 1.6.3
  role: Sparse graph operations (with torch-sparse 0.6.8 and torch-scatter 2.0.6)
    supporting the DGCN on the directed GRN
  repo: https://github.com/NWPU-903PR/DGMP
- name: Anaconda
  role: Environment manager used to create the DGMP Python 3.7 environment and install
    dependencies
- name: DiGC
  role: 'Acknowledged basis: DGMP''s template/code is borrowed from DiGC/DiGCN for
    directed graph convolutions'
  repo: https://github.com/flyingtango/DiGC
provenance:
  source_task_ids:
  - task_001
  - task_003
  source_papers:
  - doi: 10.1016/j.gpb.2022.11.004
    title: 'DGMP: Identifying Cancer Driver Genes by Jointing DGCN and MLP from Multi-Omics
      Genomic Data'
  source_content_hash: sha256:a12ca60dd43088b57712255091ee859a795aa11f7e9c7097cd2d23c933e85b23
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
  - torch-geometric 1.6.3
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_2640
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0749
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# multi-omics-feature-learning

## Summary

Train the DGMP model, which jointing a directed graph convolutional network (DGCN) with an MLP, to identify cancer driver genes by integrating multidimensional multi-omics gene features (SNVs, CNAs, DNA methylation, gene expression) with topological features of a directed gene regulatory network (GRN).

## When to use

Use when labeled driver/non-driver genes are available on a directed GRN (DawnNet, KEGG, RegNetwork) with per-gene multi-omics features, and the goal is to predict cancer driver genes via graph + feature learning.

## When NOT to use

- The network is an undirected PPI network rather than a directed GRN — DGCN is designed to capture directed regulation information
- Only single-gene features without graph topology are available and a purely feature-based classifier is intended; DGMP's design couples multi-omics features with GRN topological structure
- Patient-level prediction is required — DGMP averages omics across all samples, which ignores the specific characteristics of individual cancer patients

## Inputs

- Directed GRN dataset shipped with the DGMP repository (DawnNet, KEGG, or RegNetwork)
- Per-gene multi-omics features (SNVs, CNAs, DNA methylation, gene expression, averaged across samples)
- Cancer driver / non-driver gene labels for 5-fold cross-validation

## Outputs

- 'logits' file in './code/result/' with two-column per-gene probabilities (column 1: non-cancer driver gene, column 2: cancer driver gene)
- 'acc' file in './code/result/' holding the mean accuracy of the 5-fold cross-validation

## How to apply

Set up an Anaconda environment named DGMP with Python 3.7, install PyTorch 1.7.1 (torchvision 0.8.2, cudatoolkit 11.0) via conda, then pip-install torch-sparse 0.6.8, torch-scatter 2.0.6, and torch-geometric 1.6.3 for sparse graph operations. Load a directed GRN dataset shipped with the DGMP repository (e.g., DawnNet) together with its multi-omics gene features, where SNVs, CNAs, DNA methylation, and gene expression are averaged across all samples to yield 4-omics features per gene. Train and test DGMP by running DGMP.py with the --gpu-no and --dataset arguments; the DGCN captures directed regulation information on the GRN (expanding the receptive field to second-order neighbors) while the MLP weighs more on gene features to mitigate bias toward topological features, and three DGCN embedding vectors plus one MLP embedding vector are concatenated and fed into a fully connected neural network. Evaluate with 5-fold cross-validation over driver/non-driver labels; set --gpu-no to -1 for CPU-only execution.

## Related tools

- **DGMP** (Core model: jointing DGCN and MLP to learn multi-omics gene features and GRN topological features for cancer driver gene identification) — https://github.com/NWPU-903PR/DGMP
- **PyTorch 1.7.1** (Deep learning framework in which DGMP is developed and run) — https://github.com/NWPU-903PR/DGMP
- **torch-geometric 1.6.3** (Sparse graph operations (with torch-sparse 0.6.8 and torch-scatter 2.0.6) supporting the DGCN on the directed GRN) — https://github.com/NWPU-903PR/DGMP
- **Anaconda** (Environment manager used to create the DGMP Python 3.7 environment and install dependencies)
- **DiGC** (Acknowledged basis: DGMP's template/code is borrowed from DiGC/DiGCN for directed graph convolutions) — https://github.com/flyingtango/DiGC

## Examples

```
cd code && python DGMP.py --gpu-no 0 --dataset DawnNet
```

## Evaluation signals

- './code/result/' contains both 'logits' and 'acc' after a run
- 'logits' holds a probability for each gene in each of the two classes (non-cancer driver and cancer driver), i.e., two columns per gene
- 'acc' stores the mean accuracy over the 5 cross-validation folds
- 5-fold cross-validation AUPRC/AUROC on DawnNet should be competitive with the reported DGMP values (e.g., AUPRC 0.875, AUROC 0.889 on DawnNet), exceeding the six comparison methods
- Independent-test metrics on DawnNet/RegNetwork (AUPRC 0.136/0.115, AUROC 0.706/0.757, TPR 0.769/0.864) provide reference ranges for generalization checks

## Limitations

- Averaging SNVs, CNAs, DNA methylation, and gene expression across all samples ignores the specific characteristics of individual cancer patients
- DGMP simply concatenates the three DGCN and one MLP embedding vectors, whereas the contributions of these four feature vectors may be different
- CPU-only execution is supported but requires setting the '--gpu-no' parameter to -1; dependency versions are pinned (PyTorch 1.7.1, cudatoolkit 11.0, torch-sparse 0.6.8, torch-scatter 2.0.6, torch-geometric 1.6.3)

## Evidence

- [abstract] we propose a novel method (called DGMP) to identify cancer driver genes by jointing directed graph convolutional network (DGCN) and multilayer perceptron (MLP): "we propose a novel method (called DGMP) to identify cancer driver genes by jointing directed graph convolutional network (DGCN) and multilayer perceptron (MLP)"
- [discussion] DGMP averages the SNVs, CNAs, DNA methylation, and gene expression of all samples to obtain 4-omics features to represent the genes for each cancer: "DGMP averages the SNVs, CNAs, DNA methylation, and gene expression of all samples to obtain 4-omics features to represent the genes for each cancer"
- [discussion] Three embedding feature vectors from DGCN and one embedding feature vector from MLP are concatenated to feed into a fully connected neural network: "Three embedding feature vectors from DGCN and one embedding feature vector from MLP are concatenated to feed into a fully connected neural network"
- [methods] python DGMP.py --gpu-no 0 --dataset DawnNet: "python DGMP.py --gpu-no 0 --dataset DawnNet"
- [readme] The probability of each gene is non-cancer driver gene (the first column) or cancer driver gene (the second column) have been stored in 'logits', and the mean accuracy of 5-CV are in 'acc': "The probability of each gene is non-cancer driver gene (the first column) or cancer driver gene (the second column) have been stored in 'logits', and the mean accuracy of 5-CV are in 'acc'"
- [readme] pip install torch-sparse==0.6.8 -f https://pytorch-geometric.com/whl/torch-1.7.1+cu110.html: "pip install torch-sparse==0.6.8 -f https://pytorch-geometric.com/whl/torch-1.7.1+cu110.html"
