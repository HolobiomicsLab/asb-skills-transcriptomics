---
name: tsne-dimensionality-reduction-visualization
description: Use when you need to visually compare separability of predicted cancer/non-cancer
  driver gene classes across alternative graph-network embedding representations,
  especially for genes ranked by neighborhood discrete entropy.
when_to_use_negative:
- The goal is a quantitative performance metric (e.g., accuracy/AUC) rather than a
  qualitative visual class-separation check — use 5-fold cross-validation instead.
- Genes of interest have low neighborhood discrete entropy (neighbors similar to the
  gene), since DGCN alone already separates them well and the MLP-offset effect will
  not be visible.
- The trained model outputs in './code/result/' are unavailable — t-SNE requires the
  stored embedding vectors ZF, Zin, Zout, ZMLP, which cannot be reconstructed from
  'logits' alone.
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_2640
- http://edamontology.org/topic_0203
tools:
- name: DGMP
  role: Trained model that supplies the DGCN (ZF, Zin, Zout) and DGCN+MLP (ZMLP) embeddings
    being visualized, plus 'logits' used to rank genes by predicted class
  repo: https://github.com/NWPU-903PR/DGMP
- name: PyTorch 1.7.1
  role: Deep-learning framework in which DGMP is developed and run to obtain embeddings
    on GPU/CPU
- name: Python 3.7
  role: Environment language used to implement the entropy computation, t-SNE projection,
    and scatter plotting
- name: Anaconda
  role: Dependency configuration for the 'DGMP' environment (Python 3.7, PyTorch 1.7.1,
    torch-geometric) needed before running DGMP.py
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.gpb.2022.11.004
    title: 'DGMP: Identifying Cancer Driver Genes by Jointing DGCN and MLP from Multi-Omics
      Genomic Data'
  source_content_hash: sha256:c55ce639ac66668d445afc148f36eb6aac5f6e324899ca5b19774563d9c3fbbf
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
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_2640
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# tsne-dimensionality-reduction-visualization

## Summary

Applies t-SNE to project high-dimensional gene embeddings (DGCN-only ZF/Zin/Zout vs concatenated DGCN+MLP ZMLP) into 2D scatter panels to visually assess whether MLP concatenation improves class separation of predicted cancer vs non-cancer driver genes.

## When to use

Use when you need to visually compare separability of predicted cancer/non-cancer driver gene classes across alternative graph-network embedding representations, especially for genes ranked by neighborhood discrete entropy.

## When NOT to use

- The goal is a quantitative performance metric (e.g., accuracy/AUC) rather than a qualitative visual class-separation check — use 5-fold cross-validation instead.
- Genes of interest have low neighborhood discrete entropy (neighbors similar to the gene), since DGCN alone already separates them well and the MLP-offset effect will not be visible.
- The trained model outputs in './code/result/' are unavailable — t-SNE requires the stored embedding vectors ZF, Zin, Zout, ZMLP, which cannot be reconstructed from 'logits' alone.

## Inputs

- Trained DGMP model outputs from './code/result/' containing 'logits' (per-gene probabilities of being non-cancer/cancer driver gene) and 'acc' (5-CV mean accuracy)
- Embedding matrices ZF, Zin, Zout (DGCN) and ZMLP (DGCN+MLP concatenation) for selected genes
- DawnNet (or KEGG/RegNetwork) gene-gene regulatory network used for neighborhood discrete entropy computation

## Outputs

- 2D t-SNE coordinates for each embedding set (ZF, Zin, Zout, ZMLP) restricted to the 100 selected high-entropy genes
- t-SNE scatter panel figure 'tsne_embedding_comparison.png' with points colored by predicted class (cancer vs non-cancer driver gene)

## How to apply

First train DGMP on the chosen GRN (e.g., DawnNet via 'python DGMP.py --gpu-no 0 --dataset DawnNet') and load the model outputs ('logits' for per-gene driver probabilities, 'acc' for 5-CV mean accuracy) from './code/result/'. Compute each gene's neighborhood discrete entropy, then select the top 50 predicted cancer driver genes and top 50 predicted non-cancer driver genes with the highest entropy — these are the cases where DGCN degrades due to dissimilar neighbors. Extract embedding vectors ZF, Zin, Zout (DGCN-only) and ZMLP (DGCN+MLP concatenation) for the selected genes, apply t-SNE to project each embedding set to 2D, and plot scatter panels coloring points by predicted class; save as 'tsne_embedding_comparison.png'. The rationale is that MLP offsets DGCN's performance degradation on genes with dissimilar neighbors, so better separation in the ZMLP panel than the ZF/Zin/Zout panels demonstrates the concatenation benefit.

## Related tools

- **DGMP** (Trained model that supplies the DGCN (ZF, Zin, Zout) and DGCN+MLP (ZMLP) embeddings being visualized, plus 'logits' used to rank genes by predicted class) — https://github.com/NWPU-903PR/DGMP
- **PyTorch 1.7.1** (Deep-learning framework in which DGMP is developed and run to obtain embeddings on GPU/CPU)
- **Python 3.7** (Environment language used to implement the entropy computation, t-SNE projection, and scatter plotting)
- **Anaconda** (Dependency configuration for the 'DGMP' environment (Python 3.7, PyTorch 1.7.1, torch-geometric) needed before running DGMP.py)

## Examples

```
cd code && python DGMP.py --gpu-no 0 --dataset DawnNet  # then load './code/result/' logits and ZF/Zin/Zout/ZMLP embeddings for t-SNE
```

## Evaluation signals

- Within-class variance of cancer and non-cancer driver gene points is smaller and between-class distances are larger in the ZMLP t-SNE panel than in the ZF/Zin/Zout panels.
- Exactly the top 50 predicted cancer drivers and top 50 predicted non-cancer drivers with the highest neighborhood discrete entropy appear in each panel (100 points per panel).
- All four embedding sets (ZF, Zin, Zout, ZMLP) are projected with t-SNE and rendered in the saved figure 'tsne_embedding_comparison.png' with class coloring.
- 'logits' loaded from './code/result/' has two columns: column 1 = probability of non-cancer driver gene, column 2 = probability of cancer driver gene, consistent with the README.
- 'acc' from './code/result/' reports the mean accuracy of 5-fold cross-validation, confirming the model outputs being visualized come from a successfully trained DGMP run.

## Limitations

- DGMP's simple concatenation of the three DGCN and one MLP embedding vectors does not account for their differing contributions, which can bias the joint embedding being visualized.
- Averaging SNVs, CNAs, DNA methylation, and gene expression across all samples to form the 4-omics gene features ignores specific characteristics of individual cancer patients.
- Training/visualization requires the environment configured for sparse graphs: torch-sparse 0.6.8, torch-scatter 2.0.6, and torch-geometric 1.6.3 under torch 1.7.1; in CPU-only scenarios '--gpu-no' must be set to -1.

## Evidence

- [readme] README result description: "The output of our model is supplied at './code/result/', which consist with two variates 'logits' and 'acc'."
- [readme] README training command: "cd code
python DGMP.py --gpu-no 0 --dataset DawnNet"
- [abstract] Article abstract MLP-offset finding: "introducing MLP into DGCN can offset the performance degradation of DGCN, and jointing MLP and DGCN can effectively improve the performance of identifying cancer driver genes"
- [discussion] Article embedding concatenation step: "Three embedding feature vectors from DGCN and one embedding feature vector from MLP are concatenated to feed into a fully connected neural network"
- [discussion] Article averaging limitation: "DGMP averages the SNVs, CNAs, DNA methylation, and gene expression of all samples to obtain 4-omics features to represent the genes for each cancer"
