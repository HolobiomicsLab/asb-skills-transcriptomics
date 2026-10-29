---
name: single-cell-count-matrix-gene-filtering
description: Use when raw single-cell count matrices must be converted into descending
  expression-ranked gene-symbol sequences for autoregressive transformer models like
  tGPT.
when_to_use_negative:
- Analysis requires absolute expression levels or fold changes among genes — tGPT
  input uses rankings only and neglects fold changes
- Downstream analysis depends on low-expressing genes, which are dropped when retaining
  only top-expressing genes
- A feature/embedding table already exists and no tokenization into gene-ranking sequences
  is needed
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: tGPT
  role: Downstream autoregressive transformer that consumes the filtered, ranked gene
    sequences over a 20,706 protein-coding gene dictionary
  repo: github.com/deeplearningplus/tGPT
- name: Scanpy
  role: Load count matrices, perform cell-level QC (genes <200, mitochondrial counts
    >30%) and gene filtering
  repo: github.com/theislab/scanpy
- name: Transformers
  role: PreTrainedTokenizerFast wraps ranked gene sequences with special tokens and
    pads/truncates to max_len 64 or 128
- name: pandas
  role: Load and manipulate the count matrix and gene annotation tables during filtering
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.isci.2023.106536
    title: Generative pretraining from large-scale transcriptomes for single-cell
      deciphering
  source_content_hash: sha256:08945d7821cdcc399a855a285ff4a4d875f88728cb89d6e02cdc0c0077ad562d
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/deeplearningplus/tGPT
  tools:
  - NumPy
  - Scanpy
  - Transformers
  - pandas
  - tGPT
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.46153846153846156
n_claims: 13
---

# single-cell-count-matrix-gene-filtering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Preprocess single-cell count matrices (e.g., HCL, GSE134355) by discarding mitochondrial, ribosomal, and non-protein-coding genes against a 20,706 protein-coding gene reference list, then rank each cell's gene symbols by descending expression to produce the token sequences tGPT consumes.

## When to use

Use when raw single-cell count matrices must be converted into descending expression-ranked gene-symbol sequences for autoregressive transformer models like tGPT.

## When NOT to use

- Analysis requires absolute expression levels or fold changes among genes — tGPT input uses rankings only and neglects fold changes
- Downstream analysis depends on low-expressing genes, which are dropped when retaining only top-expressing genes
- A feature/embedding table already exists and no tokenization into gene-ranking sequences is needed

## Inputs

- Single-cell count matrix (e.g., HCL, GEO GSE134355) as h5/mtx/csv loaded via Scanpy or pandas
- 20,706 protein-coding gene reference list
- Pretrained tGPT HuggingFace tokenizer (lixiangchun/transcriptome-gpt-1024-8-16-64)

## Outputs

- Per-cell gene-symbol strings ranked by descending expression (top 62 or 126 genes)
- Tokenized ranking sequences in form [<s>, G1, G2, ..., <e>], padded/truncated to length 64 or 128
- tokenized_ranking_sequences.l file with one tokenized sequence per cell

## How to apply

Load the deposited count matrix (e.g., HCL or GEO GSE134355) with Scanpy/pandas and discard mitochondrial genes, ribosomal genes, and non-protein-coding genes using the 20,706 protein-coding gene reference list. For each cell, sort gene symbols by descending expression and retain the top 62 (or 126) expressed genes — rank-based input is used rather than actual expression levels so the model is insensitive to batch effects and normalization. Replace regex-matched '-' and '.' characters in gene symbols with underscores before mapping to vocabulary IDs via the tGPT HuggingFace tokenizer (lixiangchun/transcriptome-gpt-1024-8-16-64). Wrap each ranked sequence with special tokens in the form [<s>, G1, G2, ..., <e>] and pad/truncate to max_len of 64 (or 128) with PreTrainedTokenizerFast. Optionally pre-filter cells with fewer than 200 expressing genes or mitochondrial counts >30% for Scanpy/scVI-comparable pipelines, writing one tokenized sequence per cell to the ranking-sequences file.

## Related tools

- **tGPT** (Downstream autoregressive transformer that consumes the filtered, ranked gene sequences over a 20,706 protein-coding gene dictionary) — github.com/deeplearningplus/tGPT
- **Scanpy** (Load count matrices, perform cell-level QC (genes <200, mitochondrial counts >30%) and gene filtering) — github.com/theislab/scanpy
- **Transformers** (PreTrainedTokenizerFast wraps ranked gene sequences with special tokens and pads/truncates to max_len 64 or 128)
- **pandas** (Load and manipulate the count matrix and gene annotation tables during filtering)

## Examples

```
tokenizer = PreTrainedTokenizerFast.from_pretrained("lixiangchun/transcriptome-gpt-1024-8-16-64"); batch = tokenizer(sequences, max_length=64, truncation=True, padding=True, return_tensors="pt")
```

## Evaluation signals

- All tokenized sequence lengths fall within {64, 128} after padding/truncation
- Filtered gene set contains no mitochondrial, ribosomal, or non-protein-coding genes relative to the 20,706 protein-coding reference list
- Each sequence begins with <s> and ends with <e> special tokens in [<s>, G1, ..., <e>] form
- Gene symbols contain no '-' or '.' characters (replaced with underscores) before tokenizer lookup
- Clustering metrics from 62 top-expressing genes are comparable to those from 126 top-expressing genes (Figure S4)

## Limitations

- tGPT uses only the top-expressing genes; information specifically represented within low-expressing genes may be missed
- Gene expression rankings rather than actual expression levels mean fold changes among genes are neglected, affecting biological interpretation
- Clustering metrics from 62 top-expressing genes are only comparable (not superior) to 126 genes, so the smaller ranking may lose signal

## Evidence
