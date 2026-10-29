---
name: five-fold-cross-validation-metric-computation
description: Use when benchmarking a graph-based driver-gene predictor (e.g., DGMP
  on DawnNet/KEGG/RegNetwork) with 5-fold CV and you need per-gene probability logits
  aggregated into mean±std AUPRC/AUROC across folds.
when_to_use_negative:
- The goal is patient-level driver gene prediction rather than per-gene evaluation
  — averaging omics across all samples ignores individual cancer patient characteristics
- The network is undirected or lacks a supplied directed GRN, so the DGCN directionality
  component cannot be exercised as in the DawnNet ablation
- Only a single train/test split is available and no 5-fold cross-validation logits
  can be collected, making the fold-aggregated mean ± std comparison impossible
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0622
tools:
- name: DGMP
  role: Model under evaluation; DGMP.py trains/tests each ablation variant and writes
    5-CV logits and accuracy to ./code/result/
  repo: https://github.com/NWPU-903PR/DGMP
- name: Anaconda
  role: Dependency configuration; create the 'DGMP' environment with Python 3.7 before
    installing PyTorch and PyTorch-Geometric packages
  repo: https://www.anaconda.com/
- name: PyTorch 1.7.1
  role: Deep-learning backend for running the DGCN and MLP ablation variants under
    5-fold cross-validation
- name: torch-geometric 1.6.3
  role: Graph learning library (with torch-sparse 0.6.8 and torch-scatter 2.0.6) required
    for dealing with the sparse directed gene regulatory graph
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.gpb.2022.11.004
    title: 'DGMP: Identifying Cancer Driver Genes by Jointing DGCN and MLP from Multi-Omics
      Genomic Data'
  source_content_hash: sha256:2249dfa7283ff99128571238541dcbe438889568672d1c08781e039fb54fd78b
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
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# five-fold-cross-validation-metric-computation

## Summary

Reproduce DGMP's 5-fold cross-validation evaluation on the DawnNet multi-omics dataset: run ablated variants (DGCN-only, MLP-only, DGCN+MLP) via DGMP.py, collect per-gene driver/non-driver probability logits and mean 5-CV accuracy from './code/result/', and compute AUPRC/AUROC to confirm that jointing DGCN and MLP improves cancer driver gene identification.

## When to use

Use when benchmarking a graph-based driver-gene predictor (e.g., DGMP on DawnNet/KEGG/RegNetwork) with 5-fold CV and you need per-gene probability logits aggregated into mean±std AUPRC/AUROC across folds.

## When NOT to use

- The goal is patient-level driver gene prediction rather than per-gene evaluation — averaging omics across all samples ignores individual cancer patient characteristics
- The network is undirected or lacks a supplied directed GRN, so the DGCN directionality component cannot be exercised as in the DawnNet ablation
- Only a single train/test split is available and no 5-fold cross-validation logits can be collected, making the fold-aggregated mean ± std comparison impossible

## Inputs

- DawnNet multi-omics gene feature matrix (averaged SNVs, CNAs, DNA methylation, gene expression across samples)
- DawnNet directed gene regulatory network supplied with the DGMP repository
- DGMP.py training script with --gpu-no and --dataset parameters
- Ablation configuration toggles for DGCN/MLP/directionality components

## Outputs

- 'logits' file in './code/result/' with per-gene two-column driver/non-driver probabilities
- 'acc' file in './code/result/' with the mean accuracy of 5-CV
- Per-variant AUPRC and AUROC aggregated as mean ± standard deviation across the 5 folds
- Table 3-style ablation comparison on DawnNet

## How to apply

Create an Anaconda environment 'DGMP' with Python 3.7, install PyTorch 1.7.1, torchvision 0.8.2, cudatoolkit 11.0, then torch-sparse 0.6.8, torch-scatter 2.0.6, and torch-geometric 1.6.3 from the torch-1.7.1+cu110 wheels. Load the DawnNet multi-omics gene feature matrix and directed gene regulatory network shipped with the repository. For each ablation configuration (DGCN alone, MLP alone, DGCN+MLP, and directionality toggles), run DGMP.py with the --dataset DawnNet flag on GPU (--gpu-no 0; set -1 for CPU-only) under 5-fold cross-validation. The model writes two outputs to './code/result/': 'logits' holding each gene's probability of being non-driver (first column) or driver (second column), and 'acc' holding the mean 5-CV accuracy. Compute AUPRC and AUROC from the logits per fold and aggregate as mean ± standard deviation across folds; the expected validation anchor is that DGMP (DGCN+MLP) reaches AUPRC 0.875 on DawnNet in 5CV, 0.047 and 0.074 higher than DGCN alone and MLP alone respectively, confirming that jointing MLP with DGCN offsets DGCN's performance degradation.

## Related tools

- **DGMP** (Model under evaluation; DGMP.py trains/tests each ablation variant and writes 5-CV logits and accuracy to ./code/result/) — https://github.com/NWPU-903PR/DGMP
- **Anaconda** (Dependency configuration; create the 'DGMP' environment with Python 3.7 before installing PyTorch and PyTorch-Geometric packages) — https://www.anaconda.com/
- **PyTorch 1.7.1** (Deep-learning backend for running the DGCN and MLP ablation variants under 5-fold cross-validation)
- **torch-geometric 1.6.3** (Graph learning library (with torch-sparse 0.6.8 and torch-scatter 2.0.6) required for dealing with the sparse directed gene regulatory graph)

## Examples

```
cd code && python DGMP.py --gpu-no 0 --dataset DawnNet
```

## Evaluation signals

- DGMP (DGCN + MLP) AUPRC on DawnNet in 5CV equals 0.875, matching Table 3's ablation results
- DGCN+MLP AUPRC exceeds DGCN-alone AUPRC by 0.047 and MLP-alone AUPRC by 0.074
- 'logits' in ./code/result/ is a two-column per-gene matrix: first column non-cancer driver gene probability, second column cancer driver gene probability
- 'acc' in ./code/result/ reports the mean accuracy of 5-CV for each variant
- Per-variant AUPRC/AUROC mean ± standard deviation across the five folds reproduces the Table 3 values within numerical tolerance

## Limitations

- Averaging SNVs, CNAs, DNA methylation, and gene expression across all samples ignores specific characteristics of individual cancer patients
- DGMP simply concatenates three DGCN embedding vectors and one MLP embedding vector, whereas the contributions of these four feature vectors may be different
- The README provides no software/library version statement beyond the requirements block; in CPU computing scenarios the '--gpu-no' parameter must be set to '-1'

## Evidence

- [readme] logits and acc outputs: "The probability of each gene is non-cancer driver gene (the first column) or cancer driver gene (the second column) have been stored in 'logits'"
- [readme] mean 5-CV accuracy: "and the mean accuracy of 5-CV are in 'acc'"
- [methods] run command: "python DGMP.py --gpu-no 0 --dataset DawnNet"
- [abstract] ablation rationale: "introducing MLP into DGCN can offset the performance degradation of DGCN"
- [results] AUPRC ablation result: [evidence span withheld — anchor status: not_found]
