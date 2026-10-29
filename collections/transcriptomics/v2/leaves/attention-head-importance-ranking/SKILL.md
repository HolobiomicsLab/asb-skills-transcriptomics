---
name: attention-head-importance-ranking
description: Use when a pretrained tGPT checkpoint and per-cell gene ranking sequences
  (with cell type labels) are available and the goal is to characterize how attention
  heads behave per cell type and rank genes by attribution.
when_to_use_negative:
- The goal is clustering or UMAP visualization of tGPT features rather than attention
  interpretability — use the README's feature-extraction + Scanpy leiden/UMAP workflow
  instead
- Only raw expression counts are available and no gene expression rankings have been
  computed — tGPT requires rank-based gene expression input, not actual expression
  levels
- The model of interest is not tGPT and does not expose per-layer, per-head attention
  weights for the 8-layer, 16-head architecture
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: tGPT
  role: Pretrained 8-layer, 16-head transformer whose attention weights are captured
    and analyzed; checkpoint 'lixiangchun/transcriptome-gpt-1024-8-16-64' is pretrained
    on 62 top-expressing genes
  repo: https://github.com/deeplearningplus/tGPT
- name: Hugging Face Transformers
  role: Provides GPT2LMHeadModel and PreTrainedTokenizerFast to load the checkpoint
    and tokenize gene ranking sequences
- name: PyTorch
  role: Runs batched forward passes under torch.no_grad() on GPU to capture attention
    weights per layer and head
- name: Scanpy
  role: Supports handling of single-cell data and cell type annotations in the surrounding
    workflow (listed among the packages required by the tGPT README)
  repo: https://github.com/theislab/scanpy
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.isci.2023.106536
    title: Generative pretraining from large-scale transcriptomes for single-cell
      deciphering
  source_content_hash: sha256:da382ab5f307b0a9ceb9a0d662e11df8fe0f27fc8d00ac43ebce427c31ca30aa
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# attention-head-importance-ranking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Interpretability analysis of the tGPT transformer: per-cell-type attention head entropy and per-head importance are computed from attention weights captured across the 8-layer, 16-head tGPT model, and gene-level token attribution scores are extracted to identify cell-type-specific marker genes.

## When to use

Use when a pretrained tGPT checkpoint and per-cell gene ranking sequences (with cell type labels) are available and the goal is to characterize how attention heads behave per cell type and rank genes by attribution.

## When NOT to use

- The goal is clustering or UMAP visualization of tGPT features rather than attention interpretability — use the README's feature-extraction + Scanpy leiden/UMAP workflow instead
- Only raw expression counts are available and no gene expression rankings have been computed — tGPT requires rank-based gene expression input, not actual expression levels
- The model of interest is not tGPT and does not expose per-layer, per-head attention weights for the 8-layer, 16-head architecture

## Inputs

- Pretrained tGPT checkpoint 'lixiangchun/transcriptome-gpt-1024-8-16-64' from Hugging Face (GPT2LMHeadModel with output_hidden_states=True)
- PreTrainedTokenizerFast tokenizer file for the same checkpoint
- Per-cell top-expressing gene ranking sequences (e.g. HCA gene rankings .txt.gz, gene symbols ranked by expression)
- Cell type annotation labels per cell (e.g. celltype label file)

## Outputs

- Per-cell-type attention head entropy heatmap (PNG)
- Per-head importance heatmap (PNG)
- CSV table of per-gene token attribution scores for the 17 marker genes (CD79A, CD79B, MS4A1, MZB1, CST3, IRF7, FCER1A, CD14, FCGR3A, PPBP, PF4, NKG7, FGFBP2, GNLY, GZMA, GZMB, PRF1)

## How to apply

Load the pretrained checkpoint 'lixiangchun/transcriptome-gpt-1024-8-16-64' with GPT2LMHeadModel(output_hidden_states=True) and its PreTrainedTokenizerFast from Hugging Face Transformers, moving the model to GPU; note the README uses max_len=64, matching tGPT's training on sequences of 62 top-expressing genes. Tokenize the per-cell top-expressing gene ranking sequences (input format [<s>, G1, G2, G3, …, <e>]) paired with cell type labels, using max_length=64 with truncation and padding, then run batched forward passes under torch.no_grad() to capture attention weights for each layer and head. Aggregate the captured attentions into per-cell-type attention head entropy and per-head importance heatmaps, and compute per-gene token attribution scores from the attention distributions, extracting marker genes such as CD79A, MS4A1, CD14, FCGR3A, NKG7, FGFBP2, GNLY, GZMA, GZMB and PRF1. Render heatmaps as PNG files and save attribution scores as CSV; the rationale is that attention patterns reveal which heads and genes drive tGPT's feature representation per cell type.

## Related tools

- **tGPT** (Pretrained 8-layer, 16-head transformer whose attention weights are captured and analyzed; checkpoint 'lixiangchun/transcriptome-gpt-1024-8-16-64' is pretrained on 62 top-expressing genes) — https://github.com/deeplearningplus/tGPT
- **Hugging Face Transformers** (Provides GPT2LMHeadModel and PreTrainedTokenizerFast to load the checkpoint and tokenize gene ranking sequences)
- **PyTorch** (Runs batched forward passes under torch.no_grad() on GPU to capture attention weights per layer and head)
- **Scanpy** (Supports handling of single-cell data and cell type annotations in the surrounding workflow (listed among the packages required by the tGPT README)) — https://github.com/theislab/scanpy

## Examples

```
tokenizer = PreTrainedTokenizerFast.from_pretrained("lixiangchun/transcriptome-gpt-1024-8-16-64"); model = GPT2LMHeadModel.from_pretrained("lixiangchun/transcriptome-gpt-1024-8-16-64", output_hidden_states=True).to("cuda")
```

## Evaluation signals

- The attention head entropy and head importance heatmap PNG files exist and reproduce the pattern of Figure 3A-3C / Figure S16
- The attribution CSV contains all 17 marker gene rows (CD79A through PRF1)
- Cytotoxic T cell / NK cell marker genes NKG7, FGFBP2, PRF1, GNLY, GZMA and GZMB show high attribution scores in their cell types
- Tokenized sequences respect max_length=64 with truncation and padding, matching tGPT's training on 62 top-expressing gene sequences
- Forward passes cover the 8 transformer layers and 16 heads, with aggregation performed per annotated cell type

## Limitations

- tGPT uses only the top-expressing genes; information represented in low-expressing genes may be missed
- tGPT uses gene expression rankings but not actual expression levels, so fold changes among genes are neglected, which can affect biological interpretation
- The language modeling objective predicts gene rankings, which is not directly related to biological issues; attention-based attributions therefore require further biological validation

## Evidence

- [readme] Here, we present an approach entitled tGPT towards integration of 22.3 million single-cell transcriptomes by modeling gene expression rankings as generative pretraining task: "Here, we present an approach entitled tGPT towards integration of 22.3 million single-cell transcriptomes by modeling gene expression rankings as generative pretraining task"
- [intro] tGPT takes as input the expression rankings of top-expressing genes rather than the actual expression levels: "tGPT takes as input the expression rankings of top-expressing genes rather than the actual expression levels"
- [discussion] We define an attention-based pathway signaling score in a similar way as54: "We define an attention-based pathway signaling score in a similar way as54"
- [readme] model = GPT2LMHeadModel.from_pretrained(checkpoint,output_hidden_states = True).transformer: "model = GPT2LMHeadModel.from_pretrained(checkpoint,output_hidden_states = True).transformer"
