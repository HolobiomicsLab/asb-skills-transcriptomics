---
name: attention-head-entropy-analysis-per-cell-type
description: Use when analyzing interpretability of the pretrained tGPT transformer
  ('lixiangchun/transcriptome-gpt-1024-8-16-64') over HCA per-cell gene ranking sequences,
  to test whether cell types show distinctive attention head entropy/importance patterns
  and to attribute scores to marker genes.
when_to_use_negative:
- The goal is feature extraction or clustering of cells rather than interpretability
  — use the standard tGPT extract-features workflow (mean of last_hidden_state over
  non-pad tokens) followed by Scanpy neighbors/Leiden/UMAP instead.
- Input data are raw expression matrices rather than precomputed top-expressing gene
  rankings — rankings must first be generated, since tGPT takes gene expression rankings,
  not actual expression levels.
- The pretrained checkpoint differs from the 62-top-gene 'lixiangchun/transcriptome-gpt-1024-8-16-64'
  model (e.g., the 126-gene model), so layer/head counts and max_length=64 assumptions
  no longer hold without adjustment.
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3474
tools:
- name: tGPT
  role: Pretrained transformer model (8 layers, 16 heads) whose attention weights
    are captured for entropy, importance, and attribution analysis; checkpoint 'lixiangchun/transcriptome-gpt-1024-8-16-64'
    loaded via GPT2LMHeadModel
  repo: https://github.com/deeplearningplus/tGPT
- name: Hugging Face Transformers
  role: Provides GPT2LMHeadModel (output_hidden_states=True, output_attentions) and
    PreTrainedTokenizerFast to load the tGPT checkpoint and tokenize gene ranking
    sequences with max_length=64
- name: PyTorch
  role: Runs batched forward passes under torch.no_grad() on GPU to capture attention
    weights per layer and head
- name: Scanpy
  role: Single-cell analysis library used in the tGPT workflow for downstream handling
    of cell type annotations and related analyses
  repo: https://github.com/theislab/scanpy
- name: pandas
  role: Reads cell type label files and writes the attribution score CSV table
- name: NumPy
  role: Aggregates captured attention tensors into per-cell-type entropy and head
    importance scores
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.isci.2023.106536
    title: Generative pretraining from large-scale transcriptomes for single-cell
      deciphering
  source_content_hash: sha256:0dcbee166381a8ffa785aff0b3132b35ce156e7dad823699667c96e558f7016b
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
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# attention-head-entropy-analysis-per-cell-type

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reproduce tGPT's interpretability analysis (Figures 3A-3C / S16) by capturing per-layer, per-head attention weights from the pretrained transformer over per-cell top-expressing gene ranking sequences, aggregating them into per-cell-type attention head entropy and head importance scores, and deriving per-gene token attribution scores that highlight cell-type marker genes.

## When to use

Use when analyzing interpretability of the pretrained tGPT transformer ('lixiangchun/transcriptome-gpt-1024-8-16-64') over HCA per-cell gene ranking sequences, to test whether cell types show distinctive attention head entropy/importance patterns and to attribute scores to marker genes.

## When NOT to use

- The goal is feature extraction or clustering of cells rather than interpretability — use the standard tGPT extract-features workflow (mean of last_hidden_state over non-pad tokens) followed by Scanpy neighbors/Leiden/UMAP instead.
- Input data are raw expression matrices rather than precomputed top-expressing gene rankings — rankings must first be generated, since tGPT takes gene expression rankings, not actual expression levels.
- The pretrained checkpoint differs from the 62-top-gene 'lixiangchun/transcriptome-gpt-1024-8-16-64' model (e.g., the 126-gene model), so layer/head counts and max_length=64 assumptions no longer hold without adjustment.

## Inputs

- Per-cell top-expressing gene ranking sequences (gene symbols ranked by expression, e.g., gzip-compressed gene rankings file such as './data/Muris_gene_rankings.txt.gz' format)
- Cell type annotation labels per cell (e.g., './data/Muris_cell_labels.txt.gz' format; HCA cell type labels for this analysis)
- Pretrained tGPT checkpoint 'lixiangchun/transcriptome-gpt-1024-8-16-64' and its PreTrainedTokenizerFast from Hugging Face

## Outputs

- Per-cell-type attention head entropy heatmap (PNG)
- Per-head attention importance heatmap (PNG)
- CSV table of per-gene token attribution scores containing all 17 marker gene rows (CD79A, CD79B, MS4A1, MZB1, CST3, IRF7, FCER1A, CD14, FCGR3A, PPBP, PF4, NKG7, FGFBP2, GNLY, GZMA, GZMB, PRF1)

## How to apply

Load the pretrained tGPT checkpoint 'lixiangchun/transcriptome-gpt-1024-8-16-64' with GPT2LMHeadModel(output_hidden_states=True) and its PreTrainedTokenizerFast from Hugging Face Transformers, move to GPU, and set model.eval(). Tokenize the per-cell top-expressing gene ranking sequences (e.g., the HCA gene rankings file) paired with cell type labels, using max_length=64 with truncation and padding, matching the [<s>, G1, G2, ..., <e>] input formulation. Run batched forward passes under torch.no_grad() over the 8-layer, 16-head transformer with output_attentions to capture attention weights per layer and head. Aggregate attentions to compute per-cell-type attention head entropy and per-head importance scores, then compute per-gene token attribution scores from the attention distributions for marker genes such as CD79A, CD79B, MS4A1, MZB1, CST3, IRF7, FCER1A, CD14, FCGR3A, PPBP, PF4, NKG7, FGFBP2, GNLY, GZMA, GZMB, PRF1. Render entropy and importance heatmaps as PNG files and save the attribution table as CSV; cytotoxic/NK marker genes (NKG7, FGFBP2, PRF1, GNLY, GZMA, GZMB) should show high attribution in cytotoxic T cells and NK cells.

## Related tools

- **tGPT** (Pretrained transformer model (8 layers, 16 heads) whose attention weights are captured for entropy, importance, and attribution analysis; checkpoint 'lixiangchun/transcriptome-gpt-1024-8-16-64' loaded via GPT2LMHeadModel) — https://github.com/deeplearningplus/tGPT
- **Hugging Face Transformers** (Provides GPT2LMHeadModel (output_hidden_states=True, output_attentions) and PreTrainedTokenizerFast to load the tGPT checkpoint and tokenize gene ranking sequences with max_length=64)
- **PyTorch** (Runs batched forward passes under torch.no_grad() on GPU to capture attention weights per layer and head)
- **Scanpy** (Single-cell analysis library used in the tGPT workflow for downstream handling of cell type annotations and related analyses) — https://github.com/theislab/scanpy
- **pandas** (Reads cell type label files and writes the attribution score CSV table)
- **NumPy** (Aggregates captured attention tensors into per-cell-type entropy and head importance scores)

## Examples

```
from transformers import PreTrainedTokenizerFast, GPT2LMHeadModel; tokenizer = PreTrainedTokenizerFast.from_pretrained('lixiangchun/transcriptome-gpt-1024-8-16-64'); model = GPT2LMHeadModel.from_pretrained('lixiangchun/transcriptome-gpt-1024-8-16-64', output_hidden_states=True, output_attentions=True).transformer.to('cuda')
```

## Evaluation signals

- Both heatmap PNG files (per-cell-type attention head entropy and per-head importance) are generated and non-empty
- The attribution CSV contains exactly the 17 expected marker gene rows: CD79A, CD79B, MS4A1, MZB1, CST3, IRF7, FCER1A, CD14, FCGR3A, PPBP, PF4, NKG7, FGFBP2, GNLY, GZMA, GZMB, PRF1
- Cytotoxic/NK marker genes NKG7, FGFBP2, PRF1, GNLY, GZMA, GZMB receive high attribution scores in cytotoxic T cells and NK cells, reproducing the pattern of Figure 3A-3C / Figure S16
- Captured attention tensors have shape consistent with the 8-layer, 16-head tGPT architecture across the batched forward passes
- Tokenized sequences respect max_length=64 with truncation and padding, matching the 62 top-expressing genes plus special tokens input format

## Limitations

- tGPT uses only top-expressing genes; therefore, it may miss information that is specifically represented within low-expressing genes.
- tGPT uses gene expression rankings but not actual expression levels, so fold changes among genes are neglected, which can affect biological interpretation.
- The language modeling objective predicts gene rankings, which is not directly related to biological issues; further study is required, and attribution patterns should be interpreted cautiously.

## Evidence

- [discussion] The input to tGPT was formulated as [<s>, G1, G2, G3, …,<e>]: "The input to tGPT was formulated as [<s>, G1, G2, G3, …,<e>]"
- [intro] tGPT takes as input the expression rankings of top-expressing genes rather than the actual expression levels: "tGPT takes as input the expression rankings of top-expressing genes rather than the actual expression levels."
- [discussion] We define an attention-based pathway signaling score in a similar way as54: "We define an attention-based pathway signaling score in a similar way as54"
- [readme] tokenizer = PreTrainedTokenizerFast.from_pretrained(tokenizer_file); model = GPT2LMHeadModel.from_pretrained(checkpoint,output_hidden_states = True): "model = GPT2LMHeadModel.from_pretrained(checkpoint,output_hidden_states = True).transformer"
- [readme] batch = tokenizer(a, max_length= max_len, truncation=True, padding=True, return_tensors="pt"): "batch = tokenizer(a, max_length= max_len, truncation=True, padding=True, return_tensors="pt")"
- [discussion] First, tGPT uses only the top-expressing genes; therefore, it may miss the information that is specifically represented within the low-expressing genes.: "First, tGPT uses only the top-expressing genes; therefore, it may miss the information that is specifically represented within the low-expressing genes."
