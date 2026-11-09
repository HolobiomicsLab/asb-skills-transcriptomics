---
name: model-embedding-comparison-across-architectures
description: Use when you have learned node embeddings from two graph-based architectures
  and need to qualitatively compare class separation (e.g., predicted driver vs non-driver
  genes) via 2D t-SNE projection.
when_to_use_negative:
- When you need quantitative performance metrics rather than qualitative embedding-space
  visualization — use the stored 5-CV mean 'acc' instead
- When the analysis goal is identifying driver genes on new multi-omics data rather
  than comparing learned representations of an already-trained DGMP model
edam_operation: http://edamontology.org/operation_2930
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_2640
tools:
- name: DGMP
  role: Trained DGCN+MLP model whose per-gene logits and embedding vectors (ZF, Zin,
    Zout, ZMLP) are extracted and compared
  repo: https://github.com/NWPU-903PR/DGMP
- name: PyTorch 1.7.1
  role: Deep learning framework in which the DGMP model is trained and from which
    embeddings are extracted
- name: Anaconda
  role: Environment configuration for the DGMP Python 3.7 dependency stack
- name: Python 3.7
  role: Runtime for the DGMP pipeline and t-SNE plotting script
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.gpb.2022.11.004
    title: 'DGMP: Identifying Cancer Driver Genes by Jointing DGCN and MLP from Multi-Omics
      Genomic Data'
  source_content_hash: sha256:41a7f3b4dcb051fd2451e54a734db35511a1d8b01864ffbca8a06f09b8466bd4
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
  edam_operation: http://edamontology.org/operation_2930
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# Model embedding comparison across architectures

## Summary

Reproduces the DGMP paper's t-SNE comparison of DGCN-only embeddings (ZF, Zin, Zout) versus the DGCN+MLP embedding (ZMLP) for predicted cancer/non-cancer driver genes with high neighborhood discrete entropy, to demonstrate that MLP concatenation offsets DGCN's degradation on genes with dissimilar neighbors.

## When to use

Use when you have learned node embeddings from two graph-based architectures and need to qualitatively compare class separation (e.g., predicted driver vs non-driver genes) via 2D t-SNE projection.

## When NOT to use

- When you need quantitative performance metrics rather than qualitative embedding-space visualization — use the stored 5-CV mean 'acc' instead
- When the analysis goal is identifying driver genes on new multi-omics data rather than comparing learned representations of an already-trained DGMP model

## Inputs

- Trained DGMP model outputs in './code/result/' containing 'logits' (per-gene non-cancer/cancer driver probabilities) and 'acc' (5-CV mean accuracy)
- Embedding vectors ZF, Zin, Zout (three DGCN embeddings) and ZMLP (DGCN+MLP embedding) for genes in the DawnNet gene regulatory network

## Outputs

- 'tsne_embedding_comparison.png': t-SNE scatter panels comparing DGCN-only (ZF, Zin, Zout) vs DGCN+MLP (ZMLP) embedding distributions, points colored by predicted class
- Ranked lists of the top 50 predicted cancer and top 50 predicted non-cancer driver genes by neighborhood discrete entropy

## How to apply

Run 'python DGMP.py --gpu-no 0 --dataset DawnNet' (use --gpu-no -1 on CPU) to obtain trained model outputs in './code/result/', which store per-gene probability 'logits' and 5-CV mean 'acc'. Compute the neighborhood discrete entropy for each gene's embedding neighborhood, rank genes by entropy within each predicted class (from the two logits columns: non-cancer driver is column 1, cancer driver is column 2), and select the top 50 predicted cancer and top 50 predicted non-cancer driver genes with highest entropy. Extract ZF, Zin, Zout (DGCN-only) and ZMLP (DGCN+MLP) embedding vectors for these genes, project each set to 2D with t-SNE, and plot scatter panels colored by predicted class, saving as 'tsne_embedding_comparison.png'. The comparison is judged correct when the DGCN+MLP (ZMLP) panel shows smaller within-class variance and larger between-class distances than the DGCN-only panels, evidencing that jointing MLP and DGCN offsets the DGCN's performance degradation.

## Related tools

- **DGMP** (Trained DGCN+MLP model whose per-gene logits and embedding vectors (ZF, Zin, Zout, ZMLP) are extracted and compared) — https://github.com/NWPU-903PR/DGMP
- **PyTorch 1.7.1** (Deep learning framework in which the DGMP model is trained and from which embeddings are extracted)
- **Anaconda** (Environment configuration for the DGMP Python 3.7 dependency stack)
- **Python 3.7** (Runtime for the DGMP pipeline and t-SNE plotting script)

## Examples

```
cd code && python DGMP.py --gpu-no 0 --dataset DawnNet
```

## Evaluation signals

- The 'logits' file in './code/result/' contains two columns — first column non-cancer driver probability, second column cancer driver probability — for all genes
- Exactly 50 predicted cancer and 50 predicted non-cancer driver genes are selected as the highest-neighborhood-discrete-entropy members of each class
- In the t-SNE panels, the ZMLP (DGCN+MLP) distribution shows smaller within-class variance and larger between-class distances than the ZF/Zin/Zout (DGCN-only) distributions
- The figure is saved as 'tsne_embedding_comparison.png' with points colored by predicted class
- The 5-CV mean accuracy is available in 'acc' confirming the model was trained successfully before embedding extraction

## Limitations

- The comparison is qualitative (visual t-SNE separation); it does not itself provide a quantitative separation metric
- Results are demonstrated on DawnNet, KEGG, and RegNetwork GRNs; behavior on other networks is not established
- CPU runs require setting '--gpu-no' to '-1'; training on GPU assumes a CUDA 11.0-compatible environment

## Evidence

- [readme] The output of our model is supplied at './code/result/', which consist with two variates 'logits' and 'acc'.: "The output of our model is supplied at './code/result/', which consist with two variates 'logits' and 'acc'."
- [readme] The probability of each gene is non-cancer driver gene (the first column) or cancer driver gene (the second column) have been stored in 'logits': "The probability of each gene is non-cancer driver gene (the first column) or cancer driver gene (the second column) have been stored in 'logits', and the mean accuracy of 5-CV are in 'acc'"
- [abstract] introducing MLP into DGCN can offset the performance degradation of DGCN, and jointing MLP and DGCN can effectively improve the performance of identifying cancer driver genes: "introducing MLP into DGCN can offset the performance degradation of DGCN, and jointing MLP and DGCN can effectively improve the performance of identifying cancer driver genes"
- [readme] Note: In CPU computing scenarios, please assignment '-1' to '--gpu-no' parameter: "Note: In CPU computing scenarios, please assignment '-1' to '--gpu-no' parameter"
