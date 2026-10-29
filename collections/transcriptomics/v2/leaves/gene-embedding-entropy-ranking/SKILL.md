---
name: gene-embedding-entropy-ranking
description: Use when trained graph-embedding outputs (e.g., DGMP logits and ZF, Zin,
  Zout, ZMLP vectors) are available and you need to isolate genes with dissimilar
  neighbors to diagnose embedding class separation.
when_to_use_negative:
- When the goal is general cancer driver gene ranking or prioritization by predicted
  probability — this skill selects by neighborhood entropy, not by driver likelihood
- When only node-level omics features (SNVs, CNAs, DNA methylation, gene expression)
  are available without trained graph embeddings or a GRN to define neighborhoods
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_2640
tools:
- name: DGMP
  role: Trained model supplying the gene probability logits and the DGCN (ZF, Zin,
    Zout) and DGCN+MLP (ZMLP) embeddings whose neighborhoods are entropy-ranked
  repo: https://github.com/NWPU-903PR/DGMP
- name: Python 3.7
  role: Implementation language for entropy computation and gene selection scripts
- name: PyTorch 1.7.1
  role: Framework in which DGMP model outputs and embedding tensors are produced and
    loaded
- name: Anaconda
  role: Environment configuration (conda env 'DGMP') for reproducible dependency setup
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.gpb.2022.11.004
    title: 'DGMP: Identifying Cancer Driver Genes by Jointing DGCN and MLP from Multi-Omics
      Genomic Data'
  source_content_hash: sha256:450c8626befcc28a97362ce7df7507f2ec79389bb78fd34e4190fe203947fd9d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/NWPU-903PR/DGMP
  tools:
  - Anaconda
  - DGMP
  - PyTorch 1.7.1
  - Python
  - Python 3.7
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 15
---

# gene-embedding-entropy-ranking

## Summary

Ranks genes by neighborhood discrete entropy computed on DGMP embedding neighborhoods, then selects the top 50 predicted cancer and top 50 predicted non-cancer driver genes with highest entropy to probe where DGCN embeddings degrade and MLP embeddings compensate.

## When to use

Use when trained graph-embedding outputs (e.g., DGMP logits and ZF, Zin, Zout, ZMLP vectors) are available and you need to isolate genes with dissimilar neighbors to diagnose embedding class separation.

## When NOT to use

- When the goal is general cancer driver gene ranking or prioritization by predicted probability — this skill selects by neighborhood entropy, not by driver likelihood
- When only node-level omics features (SNVs, CNAs, DNA methylation, gene expression) are available without trained graph embeddings or a GRN to define neighborhoods

## Inputs

- Trained DGMP model output './code/result/' containing 'logits' (per-gene cancer/non-cancer driver probabilities from 5-CV) and 'acc' (mean 5-fold cross-validation accuracy)
- Learned embedding vectors ZF, Zin, Zout (DGCN) and ZMLP (DGCN+MLP) for all genes
- Directed gene regulatory network dataset (e.g., DawnNet, KEGG, or RegNetwork) used to define gene neighborhoods

## Outputs

- Per-gene neighborhood discrete entropy scores
- Ranked gene lists by entropy within each predicted class
- Selected subset: top 50 highest-entropy predicted cancer driver genes and top 50 highest-entropy predicted non-cancer driver genes

## How to apply

Load the trained DGMP model outputs from './code/result/', which contain the 'logits' variable storing per-gene probabilities of non-cancer driver gene (first column) versus cancer driver gene (second column), plus the learned embedding vectors ZF, Zin, Zout (DGCN) and ZMLP (DGCN+MLP). Compute the neighborhood discrete entropy of each gene's embedding neighborhood as a measure of neighbor dissimilarity, and rank genes by entropy within each predicted class. Select the top 50 predicted cancer driver genes and the top 50 predicted non-cancer driver genes with the highest neighborhood discrete entropy. The rationale is that DGCN performance degrades for genes whose neighbors are dissimilar, whereas jointing MLP with DGCN offsets this degradation, so the high-entropy subsets expose the contrast between DGCN-only and DGCN+MLP embeddings. Downstream (in the companion t-SNE visualization skill), these selected genes' ZF/Zin/Zout versus ZMLP vectors are compared for within-class variance and between-class distances.

## Related tools

- **DGMP** (Trained model supplying the gene probability logits and the DGCN (ZF, Zin, Zout) and DGCN+MLP (ZMLP) embeddings whose neighborhoods are entropy-ranked) — https://github.com/NWPU-903PR/DGMP
- **Python 3.7** (Implementation language for entropy computation and gene selection scripts)
- **PyTorch 1.7.1** (Framework in which DGMP model outputs and embedding tensors are produced and loaded)
- **Anaconda** (Environment configuration (conda env 'DGMP') for reproducible dependency setup)

## Examples

```
cd code && python DGMP.py --gpu-no 0 --dataset DawnNet  # then load './code/result/' logits and embeddings to compute neighborhood discrete entropy and rank genes per class
```

## Evaluation signals

- The 'logits' array is loadable from './code/result/' and its two columns correspond to non-cancer driver gene (first column) and cancer driver gene (second column) probabilities
- Exactly 50 predicted cancer driver genes and 50 predicted non-cancer driver genes are selected, each as the highest-entropy members of its class
- Entropy computation is restricted to each gene's embedding neighborhood in the GRN (e.g., DawnNet), and ranking is done within each predicted class rather than globally
- Selected subsets cover genes whose neighborhoods are dissimilar (high entropy), matching the paper's premise that DGCN degrades on such genes
- The mean 5-CV accuracy in 'acc' confirms the model outputs loaded are from a successfully trained DGMP run (e.g., 'python DGMP.py --gpu-no 0 --dataset DawnNet')

## Limitations

- DGMP averages SNVs, CNAs, DNA methylation, and gene expression across all samples, ignoring specific characteristics of individual cancer patients, which propagates into the embeddings being entropy-ranked
- DGMP simply concatenates three DGCN embedding vectors and one MLP embedding vector, so the differing contributions of the four feature vectors are not accounted for during selection or downstream analysis
- No software/library version statement beyond PyTorch 1.7.1 / Python 3.7 is provided for the entropy-ranking analysis itself

## Evidence

- [readme] README result definition: "The probability of each gene is non-cancer driver gene (the first column) or cancer driver gene (the second column) have been stored in 'logits'"
- [methods] Model invocation: "python DGMP.py --gpu-no 0 --dataset DawnNet"
- [abstract] MLP offsets DGCN degradation (abstract): "introducing MLP into DGCN can offset the performance degradation of DGCN, and jointing MLP and DGCN can effectively improve the performance of identifying cancer driver genes"
- [discussion] Embedding concatenation: "Three embedding feature vectors from DGCN and one embedding feature vector from MLP are concatenated to feed into a fully connected neural network"
- [discussion] Averaging omics limitation: "DGMP averages the SNVs, CNAs, DNA methylation, and gene expression of all samples to obtain 4-omics features to represent the genes for each cancer"
