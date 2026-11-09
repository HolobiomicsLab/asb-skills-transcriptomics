---
name: cross-validation-model-evaluation
description: Use when benchmarking a cancer driver gene prediction model on directed
  gene regulatory networks (DawnNet, KEGG, RegNetwork) with multi-omics gene features
  and you need 5-fold CV metrics (AUPRC, AUROC, accuracy) comparable to graph-based
  baselines.
when_to_use_negative:
- 'The goal is patient-level prediction: DGMP averages SNVs, CNAs, DNA methylation,
  and gene expression across all samples, which the authors note ''will ignore the
  specific characteristics of individual cancer patients'''
- 'The GRN is undirected: the DGCN component specifically captures ''the directed
  information (i.e., regulation information)'' from directed gene regulatory networks'
- You need an undirected vanilla GCN baseline only — use EMOGI-style GCN pipelines
  instead of the DGCN+MLP joint model
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_2640
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0621
tools:
- name: DGMP
  role: The DGCN+MLP model trained and evaluated under 5-fold cross-validation via
    DGMP.py
  repo: https://github.com/NWPU-903PR/DGMP
- name: PyTorch 1.7.1
  role: Deep learning framework implementing the DGCN and MLP modules
- name: torch-geometric 1.6.3
  role: Sparse graph operations library (with torch-sparse 0.6.8 and torch-scatter
    2.0.6) for handling the directed GRN
  repo: https://github.com/flyingtango/DiGC
- name: Anaconda
  role: Dependency configuration for the Python 3.7 environment with pinned package
    versions
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.gpb.2022.11.004
    title: 'DGMP: Identifying Cancer Driver Genes by Jointing DGCN and MLP from Multi-Omics
      Genomic Data'
  source_content_hash: sha256:ac54ead17d9dc29db0b9e670e36759f9604367c269c078c87ba7ac93e9245e65
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
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_2640
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0621
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5333333333333333
n_claims: 15
---

# cross-validation-model-evaluation

## Summary

Runs 5-fold cross-validation of the DGMP directed-graph-convolution + MLP cancer driver gene classifier over driver/non-driver gene labels on directed GRNs, reporting per-gene class probabilities and mean CV accuracy for benchmarking against methods like EMOGI, NRFD, DeepWalk+SVM, PageRank, HotNet2, and MutSigCV.

## When to use

Use when benchmarking a cancer driver gene prediction model on directed gene regulatory networks (DawnNet, KEGG, RegNetwork) with multi-omics gene features and you need 5-fold CV metrics (AUPRC, AUROC, accuracy) comparable to graph-based baselines.

## When NOT to use

- The goal is patient-level prediction: DGMP averages SNVs, CNAs, DNA methylation, and gene expression across all samples, which the authors note 'will ignore the specific characteristics of individual cancer patients'
- The GRN is undirected: the DGCN component specifically captures 'the directed information (i.e., regulation information)' from directed gene regulatory networks
- You need an undirected vanilla GCN baseline only — use EMOGI-style GCN pipelines instead of the DGCN+MLP joint model

## Inputs

- Directed GRN dataset shipped with the DGMP repository (DawnNet, KEGG, or RegNetwork)
- Multi-omics gene features (SNVs, CNAs, DNA methylation, gene expression averaged across samples)
- Driver/non-driver gene labels for 5-fold cross-validation splits

## Outputs

- 'logits' file in ./code/result/ with two-column per-gene probabilities (non-cancer driver gene, cancer driver gene)
- 'acc' file in ./code/result/ storing mean 5-fold cross-validation accuracy
- AUPRC/AUROC benchmark values comparable against six baseline methods

## How to apply

Create an Anaconda environment 'DGMP' with Python 3.7, install PyTorch 1.7.1 with cudatoolkit 11.0, plus torch-sparse 0.6.8, torch-scatter 2.0.6, and torch-geometric 1.6.3 (required for sparse graph operations on the directed GRN). Load the dataset shipped with the repository (e.g. DawnNet with its multi-omics gene features covering SNVs, CNAs, DNA methylation, and gene expression averaged across samples). Train and test with 'python DGMP.py --gpu-no 0 --dataset DawnNet', setting --gpu-no to -1 for CPU-only execution. The model runs 5-fold cross-validation over the driver/non-driver gene labels; results are written to './code/result/' as 'logits' (two-column per-gene probabilities: first column non-cancer driver, second column cancer driver) and 'acc' (mean accuracy over the 5 CV folds). Compare AUPRC/AUROC against the six baselines (EMOGI, NRFD, DeepWalk+SVM, PageRank, HotNet2, MutSigCV); the paper reports DGMP's DawnNet AUPRC of 0.875 and AUROC of 0.889 exceeding all baselines.

## Related tools

- **DGMP** (The DGCN+MLP model trained and evaluated under 5-fold cross-validation via DGMP.py) — https://github.com/NWPU-903PR/DGMP
- **PyTorch 1.7.1** (Deep learning framework implementing the DGCN and MLP modules)
- **torch-geometric 1.6.3** (Sparse graph operations library (with torch-sparse 0.6.8 and torch-scatter 2.0.6) for handling the directed GRN) — https://github.com/flyingtango/DiGC
- **Anaconda** (Dependency configuration for the Python 3.7 environment with pinned package versions)

## Examples

```
conda create -n DGMP python=3.7; conda activate DGMP; python DGMP.py --gpu-no 0 --dataset DawnNet
```

## Evaluation signals

- './code/result/' contains both 'logits' and 'acc' files after the run completes
- 'logits' holds a probability for each gene in each of the two classes (first column non-cancer driver, second column cancer driver)
- 'acc' stores the mean accuracy over the 5 cross-validation folds
- AUPRC/AUROC on DawnNet should be in the range of the reported values (AUPRC 0.875, AUROC 0.889), exceeding all six baselines by 0.054–0.355 (AUPRC) and 0.053–0.284 (AUROC)
- The best DGMP result is obtained on all three GRNs (DawnNet, KEGG, RegNetwork), consistent with the paper's claim of superiority over EMOGI, NRFD, DeepWalk+SVM, PageRank, HotNet2, and MutSigCV

## Limitations

- Averaging the four omics across all samples ignores individual patient-specific characteristics
- Simple concatenation of the three DGCN embedding vectors and one MLP embedding vector does not account for their differing contributions
- CPU-only execution requires setting '--gpu-no' to -1; environment must match CUDA version (e.g. cudatoolkit 11.0) or torch installation fails

## Evidence

- [readme] Training and testing our DGMP model with the following commands: "Training and testing our DGMP model with the following commands"
- [readme] results-directory-contract: "The probability of each gene is non-cancer driver gene (the first column) or cancer driver gene (the second column) have been stored in 'logits', and the mean accuracy of 5-CV are in 'acc'"
- [readme] environment-pins: "Our project is developed using PyTorch 1.7.1 in Python 3.7. 
We recommend you to use [anaconda](https://www.anaconda.com/) for dependency configuration."
- [readme] sparse-graph-rationale: "Besides, torch-scatter and torch-sparse are required for dealing with sparse graph"
- [readme] cpu-fallback: "Note: In CPU computing scenarios, please assignment '-1' to '--gpu-no' parameter"
- [methods] cv-command: "python DGMP.py --gpu-no 0 --dataset DawnNet"
- [discussion] benchmark-superiority: "The results on three networks of DawnNet, KEGG, and RegNetwork show that DGMP is superior to other existing state-of-the-art methods in identifying the cancer driver genes."
- [discussion] omics-averaging-limitation: "DGMP averages the SNVs, CNAs, DNA methylation, and gene expression of all samples to obtain 4-omics features to represent the genes for each cancer"
