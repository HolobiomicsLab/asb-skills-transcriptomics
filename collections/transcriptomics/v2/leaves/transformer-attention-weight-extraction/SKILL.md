---
name: transformer-attention-weight-extraction
description: Use when you have tokenized transcriptome ranking sequences (e.g., top-62-expressing
  gene lists) and need the raw attention weights of tGPT's 8 transformer layers to
  derive per-head entropy, head importance, or token attribution features.
when_to_use_negative:
- You only need a fixed-size feature embedding for clustering or UMAP — extract last_hidden_state
  features instead of attention weights
- The input is raw expression counts rather than expression rankings — tGPT expects
  descending top-expressing gene ranking sequences, not actual expression levels
- Your transcriptomes contain more than the 62 top-expressing genes the 64-token checkpoint
  models; the 126-gene variant or a different checkpoint is needed
edam_operation: http://edamontology.org/operation_0289
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3474
tools:
- name: tGPT
  role: Pretrained transformer whose attention weights (8 layers x 16 heads) are extracted;
    checkpoint 'lixiangchun/transcriptome-gpt-1024-8-16-64'
  repo: github.com/deeplearningplus/tGPT
- name: Hugging Face Transformers
  role: Provides PreTrainedTokenizerFast, GPT2LMHeadModel, and GPT2Model with output_attentions=True
    to capture attention matrices
- name: PyTorch
  role: Runs batched forward passes on GPU under torch.no_grad() and hosts the Dataset/DataLoader
    over gene ranking lines
- name: pandas
  role: Assembles per-patient x per-head entropy tables (attention_head_entropy.csv)
    and attribution score CSVs
- name: Scanpy
  role: Renders per-cell-type entropy/importance outputs and downstream UMAP visualization
    of extracted features
  repo: github.com/theislab/scanpy
provenance:
  source_task_ids:
  - task_008
  - task_005
  source_papers:
  - doi: 10.1016/j.isci.2023.106536
    title: Generative pretraining from large-scale transcriptomes for single-cell
      deciphering
  source_content_hash: sha256:03f619edf151c80cd4a91a2089bd16a56dd9ba40e9b47d4564f6ae6ba744f8a9
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
  - Transformers
  - pandas
  - tGPT
  edam_operation: http://edamontology.org/operation_0289
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4117647058823529
n_claims: 17
---

# transformer-attention-weight-extraction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Extracts per-layer, per-head attention weight matrices from the pretrained tGPT transformer (8 layers x 16 heads, checkpoint 'lixiangchun/transcriptome-gpt-1024-8-16-64') over gene-ranking token sequences, to compute attention head entropy, head importance, and gene token attribution scores that link model internals to cell types or clinical phenotypes such as immunotherapy response.

## When to use

Use when you have tokenized transcriptome ranking sequences (e.g., top-62-expressing gene lists) and need the raw attention weights of tGPT's 8 transformer layers to derive per-head entropy, head importance, or token attribution features.

## When NOT to use

- You only need a fixed-size feature embedding for clustering or UMAP — extract last_hidden_state features instead of attention weights
- The input is raw expression counts rather than expression rankings — tGPT expects descending top-expressing gene ranking sequences, not actual expression levels
- Your transcriptomes contain more than the 62 top-expressing genes the 64-token checkpoint models; the 126-gene variant or a different checkpoint is needed

## Inputs

- Gene ranking sequences (gene symbols ranked by descending expression, e.g., Muris_gene_rankings.txt.gz or bulk-tumor ranking files)
- Pretrained tGPT checkpoint 'lixiangchun/transcriptome-gpt-1024-8-16-64' and its PreTrainedTokenizerFast tokenizer
- Optional cell-type labels (e.g., Muris_cell_labels.txt.gz) or RECIST response annotations (25 CR, 43 PR, 63 SD, 63 PD) for grouping
- HCA gene rankings file pairing each cell with its cell type label

## Outputs

- Per-sample (patient or cell) attention weight matrices from all heads of the 8 tGPT transformer layers
- Per-patient x per-head attention entropy table (attention_head_entropy.csv)
- Per-cell-type attention head entropy and head importance heatmaps (PNG)
- Per-gene token attribution score CSV (e.g., rows for CD79A, MS4A1, CST3, IRF7, CD14, FCGR3A, PPBP, PF4, NKG7, FGFBP2, GNLY, GZMA, GZMB, PRF1, MZB1)

## How to apply

Load the tGPT checkpoint with Transformers (GPT2LMHeadModel or GPT2Model with output_attentions=True) and its PreTrainedTokenizerFast, and move the model to GPU. Convert each transcriptome (single cell or bulk tumor sample) into a descending expression-ranking sequence of top-expressing genes and tokenize with max_length=64, truncation and padding enabled. Run batched forward passes under torch.no_grad() to collect attention weight matrices from all heads of the 8 transformer layers for every sample. Downstream, compute per-head Shannon entropy and head importance per patient or cell type (assembling tables like attention_head_entropy.csv), or compute per-gene token attribution scores from the attention distributions (e.g., marker genes NKG7, FGFBP2, GNLY, GZMA, GZMB, PRF1, CD79A). Rationale: tGPT autoregressively models gene expression rankings, so attention patterns over the rank tokens serve as interpretable feature patterns associated with cell states and, for bulk urothelial carcinoma ICB data, with RECIST response groups (SD, PR, CR, PD).

## Related tools

- **tGPT** (Pretrained transformer whose attention weights (8 layers x 16 heads) are extracted; checkpoint 'lixiangchun/transcriptome-gpt-1024-8-16-64') — github.com/deeplearningplus/tGPT
- **Hugging Face Transformers** (Provides PreTrainedTokenizerFast, GPT2LMHeadModel, and GPT2Model with output_attentions=True to capture attention matrices)
- **PyTorch** (Runs batched forward passes on GPU under torch.no_grad() and hosts the Dataset/DataLoader over gene ranking lines)
- **pandas** (Assembles per-patient x per-head entropy tables (attention_head_entropy.csv) and attribution score CSVs)
- **Scanpy** (Renders per-cell-type entropy/importance outputs and downstream UMAP visualization of extracted features) — github.com/theislab/scanpy

## Examples

```
tokenizer = PreTrainedTokenizerFast.from_pretrained('lixiangchun/transcriptome-gpt-1024-8-16-64'); model = GPT2Model.from_pretrained('lixiangchun/transcriptome-gpt-1024-8-16-64', output_attentions=True).to('cuda')
```

## Evaluation signals

- Entropy table covers all 298 patients of the urothelial carcinoma ICB cohort (25 CR, 43 PR, 63 SD, 63 PD)
- Exactly 5 attention heads are flagged as significantly different between CR/PR and SD/PD groups
- Group-level per-head entropy panels reproduce the gradually varying SD→PR→CR entropy trend of Figure 5G-5N
- Attribution CSV contains all 17 marker gene rows (CD79A through PRF1) with cytotoxic markers NKG7, FGFBP2, PRF1, GNLY, GZMA, GZMB highly scored in cytotoxic T and NK cells
- Heatmap PNG files for per-cell-type head entropy and head importance exist (Figure 3A-3C / Figure S16 pattern)

## Limitations

- tGPT uses only the top-expressing genes and may miss information represented in low-expressing genes
- tGPT uses gene expression rankings but not actual expression levels, so fold changes among genes are neglected and this can affect biological interpretation
- The language modeling objective predicts gene rankings, which is not directly related to biological issues; further study is required
- Pretrained checkpoint availability beyond the code repository link is not explicitly stated in the article

## Evidence
