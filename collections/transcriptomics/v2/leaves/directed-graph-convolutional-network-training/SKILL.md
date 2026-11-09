---
name: directed-graph-convolutional-network-training
description: Use when you need to reproduce or benchmark cancer driver gene identification
  on a directed GRN with multi-omics gene features, comparing AUPRC/AUROC against
  EMOGI, NRFD, DeepWalk+SVM, PageRank, HotNet2, or MutSigCV.
when_to_use_negative:
- Your network is undirected — the DGCN is designed to capture directed regulation
  information from directed GRNs
- You need patient-specific (per-sample) predictions — DGMP averages the four omics
  layers across all samples, which ignores individual patient characteristics
- You need a method that weighs embedding contributions adaptively — DGMP simply concatenates
  three DGCN and one MLP embedding vectors without accounting for differing contributions
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_2640
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0602
tools:
- name: DGMP
  role: 'Core model: jointed DGCN and MLP that integrates multi-omics gene features
    with topological structure features of the directed GRN to identify cancer driver
    genes'
  repo: https://github.com/NWPU-903PR/DGMP
- name: PyTorch 1.7.1
  role: Deep learning framework in which DGMP is developed and trained
- name: Anaconda
  role: Environment and dependency configuration (Python 3.7 environment 'DGMP', cudatoolkit
    11.0)
- name: torch-geometric 1.6.3
  role: Sparse graph operations library (with torch-sparse 0.6.8 and torch-scatter
    2.0.6) required by the graph convolution layers
- name: DiGC
  role: External directed graph convolution reference implementation acknowledged
    by the DGMP repository
  repo: https://github.com/flyingtango/DiGC
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.gpb.2022.11.004
    title: 'DGMP: Identifying Cancer Driver Genes by Jointing DGCN and MLP from Multi-Omics
      Genomic Data'
  source_content_hash: sha256:0c2b1e8fefddf312229a13377e2c6400c416112d36c665654ac43c066a9a88ec
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
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# directed-graph-convolutional-network-training

## Summary

Train and evaluate DGMP, a directed graph convolutional network (DGCN) jointed with an MLP, to identify cancer driver genes from multi-omics features on directed gene regulatory networks (DawnNet, KEGG, RegNetwork) using 5-fold cross-validation.

## When to use

Use when you need to reproduce or benchmark cancer driver gene identification on a directed GRN with multi-omics gene features, comparing AUPRC/AUROC against EMOGI, NRFD, DeepWalk+SVM, PageRank, HotNet2, or MutSigCV.

## When NOT to use

- Your network is undirected — the DGCN is designed to capture directed regulation information from directed GRNs
- You need patient-specific (per-sample) predictions — DGMP averages the four omics layers across all samples, which ignores individual patient characteristics
- You need a method that weighs embedding contributions adaptively — DGMP simply concatenates three DGCN and one MLP embedding vectors without accounting for differing contributions

## Inputs

- Directed gene regulatory network dataset shipped with the DGMP repository (DawnNet, KEGG, or RegNetwork)
- Multi-omics gene features (SNVs, CNAs, DNA methylation, gene expression averaged across samples) for genes in the GRN
- Binary driver/non-driver gene labels for 5-fold cross-validation

## Outputs

- 'logits' file in ./code/result/ with two-column per-gene probabilities (first column: non-cancer driver gene, second column: cancer driver gene)
- 'acc' file in ./code/result/ storing the mean accuracy over the 5 cross-validation folds

## How to apply

Create an Anaconda environment 'DGMP' with Python 3.7, install PyTorch 1.7.1 with cudatoolkit 11.0, then pip install torch-sparse 0.6.8, torch-scatter 2.0.6, and torch-geometric 1.6.3 for sparse graph operations. Load the directed GRN dataset shipped with the DGMP repository (e.g., DawnNet) together with its multi-omics gene features (SNVs, CNAs, DNA methylation, gene expression averaged across samples). Train and test with 'python DGMP.py --gpu-no 0 --dataset DawnNet' (set '--gpu-no' to -1 for CPU-only execution), running 5-fold cross-validation over driver/non-driver gene labels. The DGCN captures directed regulation information from second-order neighbors while the MLP weighs gene features to mitigate bias toward topological features. Write the per-gene two-column probabilities to 'logits' and the mean 5-CV accuracy to 'acc' under './code/result/'.

## Related tools

- **DGMP** (Core model: jointed DGCN and MLP that integrates multi-omics gene features with topological structure features of the directed GRN to identify cancer driver genes) — https://github.com/NWPU-903PR/DGMP
- **PyTorch 1.7.1** (Deep learning framework in which DGMP is developed and trained)
- **Anaconda** (Environment and dependency configuration (Python 3.7 environment 'DGMP', cudatoolkit 11.0))
- **torch-geometric 1.6.3** (Sparse graph operations library (with torch-sparse 0.6.8 and torch-scatter 2.0.6) required by the graph convolution layers)
- **DiGC** (External directed graph convolution reference implementation acknowledged by the DGMP repository) — https://github.com/flyingtango/DiGC

## Examples

```
conda create -n DGMP python=3.7; conda activate DGMP; conda install pytorch==1.7.1 torchvision==0.8.2 cudatoolkit=11.0; cd code; python DGMP.py --gpu-no 0 --dataset DawnNet
```

## Evaluation signals

- './code/result/' contains both 'logits' and 'acc' after a completed run
- 'logits' holds a two-column probability for each gene in the network (column 1 = non-cancer driver, column 2 = cancer driver)
- 'acc' stores the mean accuracy over the 5 cross-validation folds
- In 5-fold cross-validation, DGMP should achieve the best AUPRC/AUROC on all three GRNs; e.g., on DawnNet AUPRC of 0.875 and AUROC of 0.889, exceeding the six comparison methods by 0.054–0.355 and 0.053–0.284 respectively
- Environment resolves cleanly: PyTorch 1.7.1 in Python 3.7 with torch-sparse 0.6.8, torch-scatter 2.0.6, and torch-geometric 1.6.3 installed against the cu110 wheel index

## Limitations

- Averaging SNVs, CNAs, DNA methylation, and gene expression across all samples ignores the specific characteristics of individual cancer patients
- Simple concatenation of the three DGCN and one MLP embedding vectors does not account for their differing contributions
- On CPU-only machines the '--gpu-no' parameter must be set to -1; the README's example assumes CUDA 11.0
- Only 5-fold cross-validation accuracy is written to 'acc'; full AUPRC/AUROC benchmark figures come from the paper's evaluation against six baselines

## Evidence

- [readme] Training and testing our DGMP model with the following commands: "Training and testing our DGMP model with the following commands:

shell
cd code
python DGMP.py --gpu-no 0 --dataset DawnNet
"
- [readme] Environment creation and dependency installation: "Our project is developed using PyTorch 1.7.1 in Python 3.7. 
We recommend you to use [anaconda](https://www.anaconda.com/) for dependency configuration."
- [readme] Output artifacts of the model: "The output of our model is supplied at './code/result/', which consist with two variates 'logits' and 'acc'."
- [intro] Model architecture rationale: "We use the DGCN model to perform the graph convolutional operation on the directed GRN for capturing the directed information (i.e., regulation information)"
- [discussion] Benchmark result on the three GRNs: "The results on three networks of DawnNet, KEGG, and RegNetwork show that DGMP is superior to other existing state-of-the-art methods in identifying the cancer driver genes."
- [discussion] Omics averaging step: "DGMP averages the SNVs, CNAs, DNA methylation, and gene expression of all samples to obtain 4-omics features to represent the genes for each cancer"
