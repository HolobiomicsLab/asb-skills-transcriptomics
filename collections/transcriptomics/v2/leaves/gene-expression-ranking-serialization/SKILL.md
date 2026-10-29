---
name: gene-expression-ranking-serialization
description: Use when raw single-cell count matrices (e.g., HCL, GSE134355) must be
  converted into token sequences for a GPT-style autoregressive transformer that models
  gene-expression rankings rather than expression values.
when_to_use_negative:
- Downstream analysis needs quantitative expression magnitudes or fold changes — ranking
  serialization discards them by design
- The biological signal of interest lives in low-expressing genes, which are dropped
  by the top-62/126-gene ranking scheme
- Input is already a feature-embedding table (e.g., hidden states from tGPT layers);
  only a count/count-derived matrix needs this serialization
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: tGPT
  role: Target autoregressive model whose input format ([<s>, G1, ..., <e>] ranked
    sequences over 20,706 protein-coding genes) this serialization produces
  repo: https://github.com/deeplearningplus/tGPT
- name: Transformers (PreTrainedTokenizerFast)
  role: Tokenizes regex-normalized ranked gene symbols into padded/truncated token-ID
    batches (max_length=64)
- name: Scanpy
  role: Loads the count matrix and applies the mitochondrial/ribosomal/non-protein-coding
    gene filter before ranking
  repo: https://github.com/theislab/scanpy
- name: PyTorch
  role: DataLoader batching (batch_size=64) and CUDA device handling for tokenization/model
    inference
- name: pandas
  role: Ingests count matrices and cell-type annotation files (e.g., Muris_cell_labels.txt.gz)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.isci.2023.106536
    title: Generative pretraining from large-scale transcriptomes for single-cell
      deciphering
  source_content_hash: sha256:1d69ac919a29f4ffe56ad2dd9381cfdf7689f9bd413127a24be89de674fd1074
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
  - Transformers
  - Transformers (PreTrainedTokenizerFast)
  - pandas
  - tGPT
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 14
---

# gene-expression-ranking-serialization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Serializes single-cell count matrices into ranked gene-symbol token sequences for tGPT-style autoregressive pretraining: each cell's protein-coding genes are ranked by descending expression, regex-normalized, mapped to a 20,706-gene vocabulary, and wrapped as [<s>, G1, G2, ..., <e>].

## When to use

Use when raw single-cell count matrices (e.g., HCL, GSE134355) must be converted into token sequences for a GPT-style autoregressive transformer that models gene-expression rankings rather than expression values.

## When NOT to use

- Downstream analysis needs quantitative expression magnitudes or fold changes — ranking serialization discards them by design
- The biological signal of interest lives in low-expressing genes, which are dropped by the top-62/126-gene ranking scheme
- Input is already a feature-embedding table (e.g., hidden states from tGPT layers); only a count/count-derived matrix needs this serialization

## Inputs

- Raw single-cell count matrix (e.g., HCL, GEO GSE134355) loadable via Scanpy/pandas
- 20,706 protein-coding gene reference list for filtering
- Gene symbols ranked by expression per cell (text file such as Muris_gene_rankings.txt.gz)
- tGPT HuggingFace tokenizer: lixiangchun/transcriptome-gpt-1024-8-16-64

## Outputs

- One tokenized ranking sequence per cell (special-token-wrapped, padded/truncated to 64 or 128 tokens)
- Token-ID tensors ready for GPT2LMHeadModel / GPT2Model feature extraction

## How to apply

Load the count matrix with Scanpy/pandas and discard mitochondrial, ribosomal, and non-protein-coding genes using the 20,706 protein-coding gene reference. For each cell, sort gene symbols by descending expression and retain the top 62 (or 126) expressed genes — the article shows clustering metrics from 62 genes are comparable to 126. Replace '-' and '.' characters in gene symbols with underscores (regex '\-|\.') and map to vocabulary IDs with the HuggingFace tokenizer lixiangchun/transcriptome-gpt-1024-8-16-64 via PreTrainedTokenizerFast. Wrap each ranked sequence as [<s>, G1, G2, ..., <e>] and pad/truncate to max_len of 64 (or 128), writing one tokenized sequence per cell for downstream feature extraction with GPT2LMHeadModel.

## Related tools

- **tGPT** (Target autoregressive model whose input format ([<s>, G1, ..., <e>] ranked sequences over 20,706 protein-coding genes) this serialization produces) — https://github.com/deeplearningplus/tGPT
- **Transformers (PreTrainedTokenizerFast)** (Tokenizes regex-normalized ranked gene symbols into padded/truncated token-ID batches (max_length=64))
- **Scanpy** (Loads the count matrix and applies the mitochondrial/ribosomal/non-protein-coding gene filter before ranking) — https://github.com/theislab/scanpy
- **PyTorch** (DataLoader batching (batch_size=64) and CUDA device handling for tokenization/model inference)
- **pandas** (Ingests count matrices and cell-type annotation files (e.g., Muris_cell_labels.txt.gz))

## Examples

```
tokenizer = PreTrainedTokenizerFast.from_pretrained("lixiangchun/transcriptome-gpt-1024-8-16-64"); batch = tokenizer([re.sub(r'\-|\.', '_', s) for s in lines], max_length=64, truncation=True, padding=True, return_tensors="pt")
```

## Evaluation signals

- Every output sequence length falls within {64, 128} after padding/truncation with PreTrainedTokenizerFast
- All mapped tokens come from the 20,706 protein-coding gene vocabulary; no mitochondrial, ribosomal, or non-protein-coding genes remain
- Gene symbols containing '-' or '.' are regex-substituted with '_' before tokenization so they resolve in the vocabulary
- Each sequence begins with <s> and ends with <e> before padding, matching the [<s>, G1, G2, G3, ..., <e>] formulation
- Downstream clustering with 62 top genes yields metrics comparable to the 126-gene variant (Figure S4 reference)

## Limitations

- tGPT uses only top-expressing genes, so information represented within low-expressing genes may be missed
- Rank-based input neglects fold changes among genes, which can affect biological interpretation
- The language-modeling objective predicts gene rankings, which is not directly related to biological issues and requires further study

## Evidence
