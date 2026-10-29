---
name: knn-graph-leiden-clustering
description: Use when cell/gene feature embeddings (e.g. from tGPT) must be clustered
  into groups and quantitatively compared with known cell type annotations using NMI,
  ARI, and FMI metrics.
when_to_use_negative:
- Input is already a cluster assignment or metric table — no feature extraction or
  graph clustering is needed
- Data has not been reduced to top-expressing gene rankings (tGPT takes expression
  rankings of top-expressing genes, not raw counts or normalized expression levels)
- No ground-truth cell type labels exist — external metrics NMI/ARI/FMI cannot be
  computed, so the grid-search evaluation component is inapplicable
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: tGPT
  role: Pretrained transformer that supplies feature representations from its embedding
    layer and 8 transformer layers for downstream KNN graph clustering
  repo: https://github.com/deeplearningplus/tGPT
- name: Scanpy
  role: Provides sc.pp.neighbors for KNN graph construction, sc.tl.leiden for clustering,
    and sc.tl.umap for visualization
  repo: https://github.com/theislab/scanpy
- name: PyTorch
  role: Backend for running the GPT2 model and DataLoader feature extraction on GPU/CPU
- name: transformers
  role: Supplies PreTrainedTokenizerFast and GPT2LMHeadModel for loading the tGPT
    checkpoint and producing last_hidden_state
- name: NumPy
  role: Stacks mean-pooled per-cell feature vectors into the feature matrix (np.stack)
- name: pandas
  role: Reads cell type label files and manages the aggregated metrics table
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.isci.2023.106536
    title: Generative pretraining from large-scale transcriptomes for single-cell
      deciphering
  source_content_hash: sha256:58baf4afe31ae10665a2c15d4ffc8a59bac2df7ae2546c8aaf4480ef6035da5a
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/deeplearningplus/tGPT
  repo_url: https://github.com/deeplearningplus/tGPT
  tools:
  - NumPy
  - PyTorch
  - Scanpy
  - pandas
  - tGPT
  - transformers
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# knn-graph-leiden-clustering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Builds a KNN neighbor graph on tGPT feature representations and clusters cells with Leiden, evaluating NMI/ARI/FMI against ground-truth cell type labels. It reproduces the clustering-performance evaluation of the tGPT transformer model across single-cell and bulk tissue datasets.

## When to use

Use when cell/gene feature embeddings (e.g. from tGPT) must be clustered into groups and quantitatively compared with known cell type annotations using NMI, ARI, and FMI metrics.

## When NOT to use

- Input is already a cluster assignment or metric table — no feature extraction or graph clustering is needed
- Data has not been reduced to top-expressing gene rankings (tGPT takes expression rankings of top-expressing genes, not raw counts or normalized expression levels)
- No ground-truth cell type labels exist — external metrics NMI/ARI/FMI cannot be computed, so the grid-search evaluation component is inapplicable

## Inputs

- Gene-ranking text files (e.g. Muris_gene_rankings.txt.gz) with gene symbols ranked by expression per cell
- Ground-truth cell type annotation file (e.g. Muris_cell_labels.txt.gz, one label per line)
- Pretrained tGPT checkpoint 'lixiangchun/transcriptome-gpt-1024-8-16-64' (Hugging Face tokenizer + GPT2LMHeadModel)

## Outputs

- Per-cell feature matrix (NumPy array, mean-pooled hidden states) stored in AnnData
- Leiden cluster assignments per cell
- clustering_metrics.csv with per-dataset NMI, ARI, and FMI for all six evaluation datasets
- UMAP visualizations colored by celltype and leiden clusters

## How to apply

Extract tGPT feature representations from the pretrained checkpoint 'lixiangchun/transcriptome-gpt-1024-8-16-64' (GPT2Model with output_hidden_states=True, max_length=64, batch_size=64, mean-pooling over non-special tokens excluding <s> and <e>). Load features into a Scanpy AnnData object with ground-truth cell type annotations stored in adata.obs as a categorical column, then compute the neighbor graph with sc.pp.neighbors. Grid-search Leiden parameters over n_neighbors from 5 to 100 (step 5) and resolution from 0.1 to 2 (step 0.2), selecting the best combination per dataset. Compute NMI, ARI, and FMI between best Leiden assignments and ground-truth labels using scikit-learn external clustering metrics, then aggregate all six datasets into clustering_metrics.csv. The paper found that feature representations from the embedding layer and 8 transformer layers feed this KNN graph clustering, and that clustering metrics from 62 top-expressing genes are comparable to 126 top-expressing genes.

## Related tools

- **tGPT** (Pretrained transformer that supplies feature representations from its embedding layer and 8 transformer layers for downstream KNN graph clustering) — https://github.com/deeplearningplus/tGPT
- **Scanpy** (Provides sc.pp.neighbors for KNN graph construction, sc.tl.leiden for clustering, and sc.tl.umap for visualization) — https://github.com/theislab/scanpy
- **PyTorch** (Backend for running the GPT2 model and DataLoader feature extraction on GPU/CPU)
- **transformers** (Supplies PreTrainedTokenizerFast and GPT2LMHeadModel for loading the tGPT checkpoint and producing last_hidden_state)
- **NumPy** (Stacks mean-pooled per-cell feature vectors into the feature matrix (np.stack))
- **pandas** (Reads cell type label files and manages the aggregated metrics table)

## Examples

```
tokenizer = PreTrainedTokenizerFast.from_pretrained('lixiangchun/transcriptome-gpt-1024-8-16-64'); model = GPT2LMHeadModel.from_pretrained(checkpoint, output_hidden_states=True).transformer; Xs = [item[1:int(eos_idxs[j]),:].mean(dim=0).tolist() for ...]; adata = sc.AnnData(np.stack(Xs)); sc.pp.neighbors(adata, n_neighbors=20); sc.tl.leiden(adata, resolution=0.6)
```

## Evaluation signals

- Computed NMI falls within reported ranges: 0.75 (HCA) to 0.90 (GTEx)
- Computed ARI falls within reported ranges: 0.53 (HCL) to 0.84 (Tabula Muris)
- Computed FMI falls within reported ranges: 0.55 (HCL) to 0.85 (Tabula Muris)
- Features are mean-pooled only over non-special tokens (positions 1 through eos index, excluding <s> and <e>)
- Clustering metrics from 62 top-expressing genes are comparable to 126 top-expressing genes, providing a cross-check on gene-set size

## Limitations

- tGPT uses only the top-expressing genes; therefore, it may miss information specifically represented within the low-expressing genes
- tGPT uses gene expression rankings but not actual expression levels; fold changes among genes are neglected, which can affect biological interpretation
- The language modeling pretraining objective is not directly related to cell clustering; the last transformer layer representation is not guaranteed to give better clustering

## Evidence
