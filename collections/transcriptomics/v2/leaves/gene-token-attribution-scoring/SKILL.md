---
name: gene-token-attribution-scoring
description: Use when you have per-cell top-expressing gene ranking sequences with
  cell type labels and want to rank genes by their attention-derived attribution scores
  to find cell-type-specific markers.
when_to_use_negative:
- Input is already a gene-by-cell expression matrix without per-cell gene rankings
  — tGPT requires rank-based input of top-expressing genes, not actual expression
  levels
- The analysis goal is clustering or trajectory inference from tGPT embeddings rather
  than marker-gene attribution via attention weights
- Low-expressing genes are the target — tGPT uses only top-expressing genes and may
  miss information represented within low-expressing genes
edam_operation: http://edamontology.org/operation_0314
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3474
tools:
- name: tGPT
  role: Pretrained transformer (8-layer, 16-head GPT2LMHeadModel) whose attention
    weights are captured to compute gene token attribution scores
  repo: github.com/deeplearningplus/tGPT
- name: Hugging Face Transformers
  role: Provides GPT2LMHeadModel and PreTrainedTokenizerFast used to load the tGPT
    checkpoint and tokenize gene ranking sequences
- name: PyTorch
  role: Runs batched forward passes under torch.no_grad() on GPU to capture attention
    weights
- name: Scanpy
  role: Supports downstream handling of per-cell feature representations and cell
    type annotations
  repo: github.com/theislab/scanpy
- name: pandas
  role: Reads cell type labels and writes the attribution score CSV table
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.isci.2023.106536
    title: Generative pretraining from large-scale transcriptomes for single-cell
      deciphering
  source_content_hash: sha256:d37f11a6a8f9703bb38eae127eb310580ccfa5aa17a0bf22892f4ed45ebe7aca
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/deeplearningplus/tGPT
  tools:
  - Hugging Face Transformers
  - NumPy
  - PyTorch
  - Scanpy
  - pandas
  - tGPT
  edam_operation: http://edamontology.org/operation_0314
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# gene-token-attribution-scoring

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes per-gene token attribution scores from the attention distributions of the pretrained tGPT transformer (checkpoint 'lixiangchun/transcriptome-gpt-1024-8-16-64') to identify cell-type-specific marker genes, e.g. NKG7, FGFBP2, PRF1, GNLY, GZMA and GZMB in cytotoxic T cells and NK cells.

## When to use

Use when you have per-cell top-expressing gene ranking sequences with cell type labels and want to rank genes by their attention-derived attribution scores to find cell-type-specific markers.

## When NOT to use

- Input is already a gene-by-cell expression matrix without per-cell gene rankings — tGPT requires rank-based input of top-expressing genes, not actual expression levels
- The analysis goal is clustering or trajectory inference from tGPT embeddings rather than marker-gene attribution via attention weights
- Low-expressing genes are the target — tGPT uses only top-expressing genes and may miss information represented within low-expressing genes

## Inputs

- Gzipped gene rankings file (e.g. ./data/Muris_gene_rankings.txt.gz) with gene symbols ranked by per-cell expression
- Cell type annotation file (e.g. ./data/Muris_cell_labels.txt.gz, one label per cell)
- Pretrained tGPT checkpoint 'lixiangchun/transcriptome-gpt-1024-8-16-64' and its PreTrainedTokenizerFast tokenizer from Hugging Face
- List of candidate marker genes to extract attribution rows for

## Outputs

- CSV table of per-gene token attribution scores (one row per marker gene per cell type)
- Heatmap PNG files of per-cell-type attention head entropy and per-head importance (reproducing Figure 3A-3C / Figure S16 pattern)

## How to apply

Load the pretrained tGPT checkpoint 'lixiangchun/transcriptome-gpt-1024-8-16-64' with GPT2LMHeadModel(output_hidden_states=True) and its PreTrainedTokenizerFast from Hugging Face Transformers, and move the model to GPU. Tokenize the per-cell top-expressing gene ranking sequences (input formulated as [<s>, G1, G2, G3, …, <e>]) paired with cell type labels, using max_length=64 with truncation and padding, matching the README workflow of replacing '-' and '.' in gene symbols with '_' via regex. Run batched forward passes (batch_size=64) under torch.no_grad() over the 8-layer, 16-head tGPT transformer to capture attention weights per layer and head. Compute per-gene token attribution scores from the attention distributions, extract rows for marker genes (CD79A, CD79B, MS4A1, MZB1, CST3, IRF7, FCER1A, CD14, FCGR3A, PPBP, PF4, NKG7, FGFBP2, GNLY, GZMA, GZMB, PRF1), and save the attribution table as CSV.

## Related tools

- **tGPT** (Pretrained transformer (8-layer, 16-head GPT2LMHeadModel) whose attention weights are captured to compute gene token attribution scores) — github.com/deeplearningplus/tGPT
- **Hugging Face Transformers** (Provides GPT2LMHeadModel and PreTrainedTokenizerFast used to load the tGPT checkpoint and tokenize gene ranking sequences)
- **PyTorch** (Runs batched forward passes under torch.no_grad() on GPU to capture attention weights)
- **Scanpy** (Supports downstream handling of per-cell feature representations and cell type annotations) — github.com/theislab/scanpy
- **pandas** (Reads cell type labels and writes the attribution score CSV table)

## Examples

```
model = GPT2LMHeadModel.from_pretrained('lixiangchun/transcriptome-gpt-1024-8-16-64', output_hidden_states=True).to(device); batch = tokenizer(lines, max_length=64, truncation=True, padding=True, return_tensors='pt')
```

## Evaluation signals

- The attribution CSV contains all 17 marker gene rows (CD79A, CD79B, MS4A1, MZB1, CST3, IRF7, FCER1A, CD14, FCGR3A, PPBP, PF4, NKG7, FGFBP2, GNLY, GZMA, GZMB, PRF1)
- Cytotoxicity genes NKG7, FGFBP2, PRF1, GNLY, GZMA and GZMB show high attribution scores in cytotoxic T cells and NK cells, matching the reported finding
- The heatmap PNG files for attention head entropy and head importance exist and reproduce the pattern of Figure 3A-3C / Figure S16
- Captured attention tensors cover all 8 transformer layers and 16 heads with max_length=64 tokenized sequences

## Limitations

- tGPT uses only the top-expressing genes; therefore, it may miss information that is specifically represented within the low-expressing genes
- tGPT uses gene expression rankings but not actual expression levels, so fold changes among genes are neglected and this can affect biological interpretation
- The language modeling objective predicts gene rankings, which is not directly related to biological issues; attribution-based interpretation requires further study
- Pretrained tGPT checkpoint availability for reuse is stated only via the code repository link and the Hugging Face identifier

## Evidence

- [other] Tokenize per-cell top-expressing gene ranking sequences with max_length=64 and capture attention weights: [evidence span withheld — anchor status: not_found]
- [other] Cell-type-specific genes receive higher attribution scores in tGPT's feature representation: [evidence span withheld — anchor status: not_found]
## Pretrained model with sequence of 62 top expressing genes.
checkpoint = "lixiangchun/transcriptome-gpt-1024-8-16-64""
