---
name: attention-heatmap-visualization
description: Use when you have per-cell top-expressing gene ranking sequences with
  cell type labels and want to probe which transformer attention heads and genes drive
  tGPT's feature representation for each cell type.
when_to_use_negative:
- Input is already a feature embedding table for clustering/UMAP — use the tGPT extract-features
  workflow with Scanpy (sc.pp.neighbors, leiden, umap) instead of attention capture.
- The goal is batch-effect integration benchmarking against Scanpy, Pegasus, scVI,
  MNN, ComBat, Harmony etc. rather than interpreting attention heads or gene attributions.
- You need quantitative expression-level analysis — tGPT consumes gene expression
  rankings, not actual expression levels.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: tGPT
  role: Pretrained transformer model (8 layers, 16 heads) whose attention weights
    are captured and analyzed per cell type
  repo: https://github.com/deeplearningplus/tGPT
- name: Hugging Face Transformers
  role: Provides GPT2LMHeadModel (output_hidden_states=True) and PreTrainedTokenizerFast
    to load the tGPT checkpoint and tokenizer
- name: PyTorch
  role: Runs batched forward passes under torch.no_grad() on GPU to capture attention
    weights per layer and head
- name: NumPy
  role: Aggregates captured attention tensors into per-cell-type entropy and importance
    matrices
- name: pandas
  role: Saves the per-gene token attribution scores as a CSV table for the 17 marker
    genes
- name: Scanpy
  role: Downstream visualization context for tGPT-derived features (neighbors, leiden,
    umap) complementary to attention analysis
  repo: https://github.com/theislab/scanpy
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.isci.2023.106536
    title: Generative pretraining from large-scale transcriptomes for single-cell
      deciphering
  source_content_hash: sha256:c25a34592c11fbdf8e73f0ef9b88b4c4454cdb85b4dded8a99cd581edac087df
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
  - Hugging Face Transformers
  - NumPy
  - PyTorch
  - Scanpy
  - pandas
  - tGPT
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# attention-heatmap-visualization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Interprets the tGPT transformer (8 layers, 16 heads) by computing per-cell-type attention head entropy and head importance from captured attention weights, then extracting per-gene token attribution scores to identify cell-type-specific marker genes, rendered as heatmap PNGs and an attribution CSV.

## When to use

Use when you have per-cell top-expressing gene ranking sequences with cell type labels and want to probe which transformer attention heads and genes drive tGPT's feature representation for each cell type.

## When NOT to use

- Input is already a feature embedding table for clustering/UMAP — use the tGPT extract-features workflow with Scanpy (sc.pp.neighbors, leiden, umap) instead of attention capture.
- The goal is batch-effect integration benchmarking against Scanpy, Pegasus, scVI, MNN, ComBat, Harmony etc. rather than interpreting attention heads or gene attributions.
- You need quantitative expression-level analysis — tGPT consumes gene expression rankings, not actual expression levels.

## Inputs

- HCA gene rankings file of per-cell top-expressing gene symbols ranked by expression (e.g. gzipped text lines)
- Cell type annotation table pairing each cell with its label (e.g. cell labels text file)
- Pretrained tGPT checkpoint 'lixiangchun/transcriptome-gpt-1024-8-16-64' and its PreTrainedTokenizerFast tokenizer from Hugging Face
- List of marker genes for attribution extraction (CD79A, CD79B, MS4A1, MZB1, CST3, IRF7, FCER1A, CD14, FCGR3A, PPBP, PF4, NKG7, FGFBP2, GNLY, GZMA, GZMB, PRF1)

## Outputs

- Per-cell-type attention head entropy heatmap PNG
- Per-head attention importance heatmap PNG
- CSV table of per-gene token attribution scores containing all 17 marker gene rows

## How to apply

Load the pretrained tGPT checkpoint 'lixiangchun/transcriptome-gpt-1024-8-16-64' via GPT2LMHeadModel (output_hidden_states=True) with its PreTrainedTokenizerFast from Hugging Face Transformers and move it to GPU. Tokenize the per-cell top-expressing gene ranking sequences (input formulated as [<s>, G1, G2, G3, …,<e>]) with max_length=64, truncation=True and padding, pairing each cell with its HCA cell type label. Run batched forward passes (README uses batch_size=64) under torch.no_grad() with output_attentions enabled to capture attention over the 8-layer, 16-head transformer. Aggregate the attentions into per-cell-type attention head entropy and per-head importance matrices, then compute per-gene token attribution scores from the attention distributions. Render the entropy and importance heatmaps as PNG files (reproducing Figure 3A–3C / Figure S16) and save the attribution scores as a CSV with rows for the 17 marker genes CD79A, CD79B, MS4A1, MZB1, CST3, IRF7, FCER1A, CD14, FCGR3A, PPBP, PF4, NKG7, FGFBP2, GNLY, GZMA, GZMB, PRF1.

## Related tools

- **tGPT** (Pretrained transformer model (8 layers, 16 heads) whose attention weights are captured and analyzed per cell type) — https://github.com/deeplearningplus/tGPT
- **Hugging Face Transformers** (Provides GPT2LMHeadModel (output_hidden_states=True) and PreTrainedTokenizerFast to load the tGPT checkpoint and tokenizer)
- **PyTorch** (Runs batched forward passes under torch.no_grad() on GPU to capture attention weights per layer and head)
- **NumPy** (Aggregates captured attention tensors into per-cell-type entropy and importance matrices)
- **pandas** (Saves the per-gene token attribution scores as a CSV table for the 17 marker genes)
- **Scanpy** (Downstream visualization context for tGPT-derived features (neighbors, leiden, umap) complementary to attention analysis) — https://github.com/theislab/scanpy

## Examples

```
from transformers import GPT2LMHeadModel, PreTrainedTokenizerFast; model = GPT2LMHeadModel.from_pretrained('lixiangchun/transcriptome-gpt-1024-8-16-64', output_hidden_states=True, output_attentions=True); tok = PreTrainedTokenizerFast.from_pretrained('lixiangchun/transcriptome-gpt-1024-8-16-64')
```

## Evaluation signals

- The attention head entropy and head importance heatmap PNG files exist and visually reproduce the pattern of Figure 3A–3C / Figure S16.
- The attribution CSV contains all 17 marker gene rows (CD79A through PRF1) with finite attribution scores.
- Cytotoxic T cell and NK cell markers NKG7, FGFBP2, PRF1, GNLY, GZMA and GZMB receive high attribution scores, while B cell markers (CD79A, MS4A1) score higher in their respective cell types.
- Captured attention tensors have the expected shape of the tGPT architecture: 8 transformer layers × 16 heads per token position.
- Tokenized sequences honor max_length=64 with truncation and padding, matching the 62-top-expressing-genes checkpoint.

## Limitations

- tGPT uses only the top-expressing genes; information specifically represented within low-expressing genes may be missed.
- tGPT uses gene expression rankings but not actual expression levels, so fold changes among genes are neglected, which can affect biological interpretation.
- The language modeling objective predicts gene rankings, which is not directly related to biological issues; attribution patterns require further biological validation.
- Pretrained tGPT checkpoint availability for reuse beyond the code repository link is not explicitly stated in the article.

## Evidence
