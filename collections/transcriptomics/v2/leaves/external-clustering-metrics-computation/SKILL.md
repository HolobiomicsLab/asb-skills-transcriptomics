---
name: external-clustering-metrics-computation
description: Use when evaluating how well unsupervised clusters derived from tGPT
  (or similar) embeddings match known cell type annotations, and you need quantitative
  NMI/ARI/FMI values reproducible as a clustering_metrics.csv table.
when_to_use_negative:
- Ground-truth cell labels are absent — external metrics (NMI, ARI, FMI) require reference
  annotations and cannot be computed
- Input is already a feature/embedding table with clusters assigned — re-extracting
  tGPT representations is unnecessary; go straight to sc.pp.neighbors and metric computation
- Evaluating batch-effect sensitivity or trajectory/pseudo-time inference rather than
  clustering concordance — the paper uses separate benchmarking and Pegasus-based
  diffusion pseudo-time workflows for those tasks
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: tGPT
  role: Pretrained transformer providing feature representations whose Leiden clusters
    are scored against cell labels
  repo: https://github.com/deeplearningplus/tGPT
- name: Scanpy
  role: AnnData construction, KNN neighbor graph (sc.pp.neighbors), and Leiden clustering
    (sc.tl.leiden)
  repo: https://github.com/theislab/scanpy
- name: PyTorch / transformers
  role: Loading the GPT2 checkpoint and computing last_hidden_state embeddings with
    mean-pooling over non-special tokens
- name: scikit-learn
  role: 'External clustering metrics: NMI, ARI, and FMI between Leiden assignments
    and ground-truth labels'
- name: pandas / NumPy
  role: Stacking pooled features into a feature matrix and aggregating per-dataset
    metrics into clustering_metrics.csv
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.isci.2023.106536
    title: Generative pretraining from large-scale transcriptomes for single-cell
      deciphering
  source_content_hash: sha256:13108962fe2a743b067b132c10f84bfd32459ec45dc36ef17811035e23243252
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
  - PyTorch / transformers
  - Scanpy
  - pandas
  - pandas / NumPy
  - scikit-learn
  - tGPT
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# external-clustering-metrics-computation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes external clustering performance metrics (NMI, ARI, FMI) comparing Leiden cluster assignments of tGPT feature embeddings against ground-truth cell type labels, following the tGPT paper's grid-searched KNN-graph clustering protocol across six single-cell and bulk evaluation datasets.

## When to use

Use when evaluating how well unsupervised clusters derived from tGPT (or similar) embeddings match known cell type annotations, and you need quantitative NMI/ARI/FMI values reproducible as a clustering_metrics.csv table.

## When NOT to use

- Ground-truth cell labels are absent — external metrics (NMI, ARI, FMI) require reference annotations and cannot be computed
- Input is already a feature/embedding table with clusters assigned — re-extracting tGPT representations is unnecessary; go straight to sc.pp.neighbors and metric computation
- Evaluating batch-effect sensitivity or trajectory/pseudo-time inference rather than clustering concordance — the paper uses separate benchmarking and Pegasus-based diffusion pseudo-time workflows for those tasks

## Inputs

- Gene-ranking text files (e.g. Muris_gene_rankings.txt.gz) with gene symbols ranked descending by expression, one cell per line
- Ground-truth cell type annotation file (e.g. Muris_cell_labels.txt.gz), one label per cell
- Pretrained tGPT checkpoint 'lixiangchun/transcriptome-gpt-1024-8-16-64' via Hugging Face transformers
- Extracted feature matrix (n_cells x 1024) from the tGPT transformer hidden states

## Outputs

- Leiden cluster assignments per cell for each (n_neighbors, resolution) grid combination
- Per-dataset NMI, ARI, and FMI values comparing Leiden clusters to ground-truth cell labels
- clustering_metrics.csv aggregating metrics for all six evaluation datasets

## How to apply

Extract tGPT feature representations from gene-ranking sequences using the pretrained checkpoint 'lixiangchun/transcriptome-gpt-1024-8-16-64' (GPT2Model with output_hidden_states=True, max_length=64, batch_size=64), mean-pooling last_hidden_state over non-special tokens between the start token and the EOS index computed from the attention mask. Load the pooled features into a Scanpy AnnData object and attach ground-truth labels as a categorical in adata.obs['celltype']. Build the KNN neighbor graph with sc.pp.neighbors, then grid-search Leiden parameters over n_neighbors from 5 to 100 (step 5) and resolution from 0.1 to 2 (step 0.2), selecting the combination that best matches the ground-truth labels per dataset. Compute normalized mutual information (NMI), adjusted Rand index (ARI), and Fowlkes–Mallows index (FMI) between best Leiden assignments and true labels using scikit-learn external clustering metrics, then aggregate all six datasets into clustering_metrics.csv.

## Related tools

- **tGPT** (Pretrained transformer providing feature representations whose Leiden clusters are scored against cell labels) — https://github.com/deeplearningplus/tGPT
- **Scanpy** (AnnData construction, KNN neighbor graph (sc.pp.neighbors), and Leiden clustering (sc.tl.leiden)) — https://github.com/theislab/scanpy
- **PyTorch / transformers** (Loading the GPT2 checkpoint and computing last_hidden_state embeddings with mean-pooling over non-special tokens)
- **scikit-learn** (External clustering metrics: NMI, ARI, and FMI between Leiden assignments and ground-truth labels)
- **pandas / NumPy** (Stacking pooled features into a feature matrix and aggregating per-dataset metrics into clustering_metrics.csv)

## Examples

```
sc.pp.neighbors(adata,n_neighbors=20); sc.tl.leiden(adata,resolution=0.6); from sklearn.metrics import normalized_mutual_info_score, adjusted_rand_score, fowlkes_mallows_score
```

## Evaluation signals

- NMI falls within the reported range: 0.75 (HCA) to 0.90 (GTEx) across the six datasets
- ARI falls within the reported range: 0.53 (HCL) to 0.84 (Tabula Muris)
- FMI falls within the reported range: 0.55 (HCL) to 0.85 (Tabula Muris)
- Grid search confirms the optimum lies inside the stated search space (n_neighbors 5–100 step 5; resolution 0.1–2 step 0.2); if the optimum touches a boundary, extend the search
- clustering_metrics.csv contains one row per evaluation dataset with non-empty NMI, ARI, and FMI columns

## Limitations

- tGPT uses only top-expressing genes (62 or 126), so information carried by low-expressing genes is missed and fold changes among genes are neglected, which can affect biological interpretation
- The language modeling pretraining objective is not directly related to cell clustering, so last-layer representations are not guaranteed to give the best clustering; the paper notes features from the embedding layer and all 8 transformer layers may need to be compared
- Clustering metrics obtained from 62 top-expressing genes are only reported as comparable to 126 genes — gene-panel choice is a sensitivity point rather than a guaranteed invariant

## Evidence
