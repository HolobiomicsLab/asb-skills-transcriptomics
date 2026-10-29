---
name: single-cell-feature-extraction-from-gene-rankings
description: Use when input is ranked gene symbol lists per cell (top-expressing genes)
  rather than an expression matrix, and you need cell embeddings for clustering, batch-invariant
  integration, or UMAP visualization.
when_to_use_negative:
- Input is already a numeric expression matrix or precomputed feature table (tGPT
  expects gene-ranking sequences, not raw counts).
- Analysis depends on actual expression levels or fold changes among genes — tGPT
  uses rankings only and neglects fold changes.
- Information specifically represented in low-expressing genes is critical, since
  tGPT uses only the top-expressing genes.
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3474
tools:
- name: tGPT
  role: Pretrained transformer that autoregressively models gene-expression rankings
    and supplies the GPT-2 checkpoint used to extract cell embeddings
  repo: https://github.com/deeplearningplus/tGPT
- name: transformers
  role: Provides PreTrainedTokenizerFast and GPT2LMHeadModel (GPT2Model) for tokenizing
    rankings and running the checkpoint with output_hidden_states=True
- name: PyTorch
  role: Backend for model inference via torch.no_grad() and GPU device handling; README
    tested with torch==1.7.1
- name: Scanpy
  role: Wraps extracted features in AnnData and computes the KNN neighbor graph with
    sc.pp.neighbors (e.g. n_neighbors=20) plus Leiden clustering and UMAP; README
    tested with scanpy==1.9.1
  repo: https://github.com/theislab/scanpy
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.isci.2023.106536
    title: Generative pretraining from large-scale transcriptomes for single-cell
      deciphering
  source_content_hash: sha256:0da1725e121867eed26fdc50e42ed0e743897ae75373536856d3c9a72db376d4
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
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 21
---

# Single-cell feature extraction from gene rankings with a pretrained transformer (tGPT)

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Extracts cell-level feature embeddings from gene-expression ranking sequences using a pretrained tGPT GPT-2-style transformer (checkpoint 'lixiangchun/transcriptome-gpt-1024-8-16-64'), yielding dense vectors suitable for KNN/Leiden clustering and downstream single-cell analysis.

## When to use

Use when input is ranked gene symbol lists per cell (top-expressing genes) rather than an expression matrix, and you need cell embeddings for clustering, batch-invariant integration, or UMAP visualization.

## When NOT to use

- Input is already a numeric expression matrix or precomputed feature table (tGPT expects gene-ranking sequences, not raw counts).
- Analysis depends on actual expression levels or fold changes among genes — tGPT uses rankings only and neglects fold changes.
- Information specifically represented in low-expressing genes is critical, since tGPT uses only the top-expressing genes.

## Inputs

- Gzipped text file of gene symbols ranked by expression per cell (one ranking per line, e.g. ./data/Muris_gene_rankings.txt.gz)
- Optional gzipped/CSV cell type annotation file (e.g. ./data/Muris_cell_labels.txt.gz)
- Pretrained Hugging Face checkpoint 'lixiangchun/transcriptome-gpt-1024-8-16-64'

## Outputs

- NumPy feature matrix (n_cells x embedding_dim) of mean-pooled tGPT embeddings
- Scanpy AnnData object with embeddings in .X and celltype as categorical .obs['celltype']
- Leiden cluster assignments and UMAP embeddings for visualization

## How to apply

Build a LineDataset from the gzipped gene-ranking file, applying regex substitution of '-' and '.' with '_' so tokens match the pretrained vocabulary. Tokenize with PreTrainedTokenizerFast from the checkpoint 'lixiangchun/transcriptome-gpt-1024-8-16-64' (62 top-expressing genes; max_length=64 with truncation and padding), run GPT2LMHeadModel in eval mode with output_hidden_states on GPU in batches of 64, and mean-pool last_hidden_state over non-special token positions 1..(attention_mask.sum-1) to get one 64-dim embedding per cell. The rank-based input makes features insensitive to batch effects and normalization since tGPT models gene expression rankings, not actual levels. Stack embeddings into a NumPy array, wrap in a Scanpy AnnData, and add ground-truth cell type labels from the cell-annotation file as categorical obs['celltype'] before computing sc.pp.neighbors for clustering/UMAP.

## Related tools

- **tGPT** (Pretrained transformer that autoregressively models gene-expression rankings and supplies the GPT-2 checkpoint used to extract cell embeddings) — https://github.com/deeplearningplus/tGPT
- **transformers** (Provides PreTrainedTokenizerFast and GPT2LMHeadModel (GPT2Model) for tokenizing rankings and running the checkpoint with output_hidden_states=True)
- **PyTorch** (Backend for model inference via torch.no_grad() and GPU device handling; README tested with torch==1.7.1)
- **Scanpy** (Wraps extracted features in AnnData and computes the KNN neighbor graph with sc.pp.neighbors (e.g. n_neighbors=20) plus Leiden clustering and UMAP; README tested with scanpy==1.9.1) — https://github.com/theislab/scanpy

## Examples

```
tokenizer = PreTrainedTokenizerFast.from_pretrained('lixiangchun/transcriptome-gpt-1024-8-16-64'); model = GPT2LMHeadModel.from_pretrained('lixiangchun/transcriptome-gpt-1024-8-16-64', output_hidden_states=True).transformer.to(device)
```

## Evaluation signals

- Mean-pooled embedding dimension matches the hidden size of the checkpoint and the number of rows equals the number of input ranking lines.
- UMAP colored by 'celltype' and by 'leiden' clusters show visually aligned cluster structure (as in the README's example figures).
- Downstream clustering metrics fall within reported ranges: NMI 0.75 (HCA) to 0.90 (GTEx), ARI 0.53 (HCL) to 0.84 (Tabula Muris), FMI 0.55 (HCL) to 0.85 (Tabula Muris).
- Clustering metrics from 62 top-expressing genes are comparable to those from 126 top-expressing genes (Figure S4), so checkpoint width should not drastically change results.
- Padding/truncation to max_length=64 correctly excludes special tokens <s>/<e> from pooling via eos_idxs from batch.attention_mask.

## Limitations

- tGPT uses only the top-expressing genes and may miss information specifically represented within low-expressing genes.
- tGPT uses expression rankings, not actual expression levels; fold changes among genes are neglected, affecting biological interpretation.
- The language-modeling pretraining objective predicts gene rankings and is not directly related to cell clustering, so the last transformer layer's representation is not guaranteed to give the best clustering.
- Gene symbols containing '-' or '.' must be regex-substituted with '_' or tokenization against the pretrained vocabulary will fail.

## Evidence
