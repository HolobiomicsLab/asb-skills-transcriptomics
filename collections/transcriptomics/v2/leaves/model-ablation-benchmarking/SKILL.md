---
name: model-ablation-benchmarking
description: Use when a paper proposes a composite neural architecture (e.g., DGCN
  + MLP) and you must verify that each proposed component (directional convolution,
  MLP branch, multi-omics features) contributes to final task performance.
when_to_use_negative:
- The goal is comparing DGMP against external baselines rather than decomposing its
  own components — use a state-of-the-art comparison benchmark instead
- The task uses undirected PPI networks or patient-level features rather than the
  directed GRN with sample-averaged multi-omics gene features
- Only a single model configuration is available with no code path to toggle components
  (DGCN/MLP/directionality), making true ablation infeasible
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_2640
tools:
- name: DGMP
  role: Model under ablation; DGMP.py is run once per ablated variant with --dataset
    DawnNet and component toggles
  repo: https://github.com/NWPU-903PR/DGMP
- name: PyTorch 1.7.1
  role: Deep learning framework in which all ablated variants are trained
- name: torch-geometric 1.6.3
  role: Sparse graph operations (with torch-sparse 0.6.8, torch-scatter 2.0.6) required
    by the DGCN component
  repo: https://github.com/flyingtango/DiGC
- name: Anaconda
  role: Dependency environment ('DGMP', Python 3.7) ensuring reproducible identical
    stacks across variants
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.gpb.2022.11.004
    title: 'DGMP: Identifying Cancer Driver Genes by Jointing DGCN and MLP from Multi-Omics
      Genomic Data'
  source_content_hash: sha256:ade2c83f5579dab3780f60e7d4d0fc6f73096841f39f9508dd279c3563d350e6
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
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 14
---

# model-ablation-benchmarking

## Summary

Benchmark ablated variants of a deep learning architecture (e.g., DGMP with and without DGCN, MLP, or directional components) under identical cross-validation settings to quantify each component's contribution, using AUPRC/AUROC computed from per-gene logits.

## When to use

Use when a paper proposes a composite neural architecture (e.g., DGCN + MLP) and you must verify that each proposed component (directional convolution, MLP branch, multi-omics features) contributes to final task performance.

## When NOT to use

- The goal is comparing DGMP against external baselines rather than decomposing its own components — use a state-of-the-art comparison benchmark instead
- The task uses undirected PPI networks or patient-level features rather than the directed GRN with sample-averaged multi-omics gene features
- Only a single model configuration is available with no code path to toggle components (DGCN/MLP/directionality), making true ablation infeasible

## Inputs

- Directed gene regulatory network (DawnNet) supplied with the DGMP repository
- Multi-omics gene feature matrix (SNVs, CNAs, DNA methylation, gene expression averaged across samples)
- DGMP training script (DGMP.py) with --gpu-no and --dataset flags

## Outputs

- Per-gene driver/non-driver probability logits ('logits') for each ablated variant
- Mean 5-fold CV accuracy ('acc') per variant
- Per-variant AUPRC and AUROC aggregated as mean ± standard deviation across folds

## How to apply

Set up the exact software stack specified by the authors — here an Anaconda environment with Python 3.7, PyTorch 1.7.1, and torch-sparse 0.6.8 / torch-scatter 2.0.6 / torch-geometric 1.6.3 built for torch-1.7.1+cu110, since sparse graph ops are version-sensitive. Load the same network (DawnNet) and multi-omics gene feature matrix for every variant so only the ablated component changes. Train each variant with 5-fold cross-validation (5CV) via DGMP.py, toggling DGCN/MLP/directionality flags, using --gpu-no 0 on GPU or -1 for CPU. Collect the per-gene driver/non-driver probability logits and mean CV accuracy written to './code/result/'. Compute AUPRC and AUROC from the logits per variant and aggregate mean ± std across folds. Judge success by comparing per-variant AUPRC/AUROC to the paper's ablation table (Table 3): on DawnNet in 5CV, DGMP (DGCN + MLP) reaches AUPRC 0.875, which is 0.047 higher than DGCN alone and 0.074 higher than MLP alone, confirming that jointing DGCN and MLP improves cancer driver gene identification.

## Related tools

- **DGMP** (Model under ablation; DGMP.py is run once per ablated variant with --dataset DawnNet and component toggles) — https://github.com/NWPU-903PR/DGMP
- **PyTorch 1.7.1** (Deep learning framework in which all ablated variants are trained)
- **torch-geometric 1.6.3** (Sparse graph operations (with torch-sparse 0.6.8, torch-scatter 2.0.6) required by the DGCN component) — https://github.com/flyingtango/DiGC
- **Anaconda** (Dependency environment ('DGMP', Python 3.7) ensuring reproducible identical stacks across variants)

## Examples

```
conda activate DGMP; cd code; python DGMP.py --gpu-no 0 --dataset DawnNet
```

## Evaluation signals

- Per-variant AUPRC/AUROC mean ± std across 5CV folds matches the Table 3 ablation results exactly
- On DawnNet 5CV, DGMP (DGCN + MLP) AUPRC = 0.875, exceeding DGCN alone by 0.047 and MLP alone by 0.074
- './code/result/' contains both 'logits' (two columns: non-driver vs driver gene probabilities) and 'acc' (mean 5-CV accuracy) for each variant
- All variants are trained on the same DawnNet network and multi-omics feature matrix with identical 5-fold CV splits
- GPU run uses --gpu-no 0; CPU fallback uses --gpu-no -1 and yields consistent results

## Limitations

- Averaging SNVs, CNAs, DNA methylation, and gene expression across all samples ignores specific characteristics of individual cancer patients
- Simple concatenation of three DGCN embedding vectors and one MLP embedding vector does not account for their differing contributions
- Results are network-specific (DawnNet, KEGG, RegNetwork); ablation conclusions may not transfer to other GRNs or cancer types

## Evidence

- [results] Ablation finding on DawnNet 5CV: [evidence span withheld — anchor status: not_found]
- [abstract] Paper claim on MLP offsetting DGCN degradation: "The ablation experimental results on the DawnNet network indicate that introducing MLP into DGCN can offset the performance degradation of DGCN"
- [readme] README invocation and output location: "python DGMP.py --gpu-no 0 --dataset DawnNet"
- [readme] README output artifacts: "The output of our model is supplied at './code/result/', which consist with two variates 'logits' and 'acc'."
- [readme] Dependency pinning for sparse graph ops: "pip install torch-sparse==0.6.8 -f https://pytorch-geometric.com/whl/torch-1.7.1+cu110.html"
