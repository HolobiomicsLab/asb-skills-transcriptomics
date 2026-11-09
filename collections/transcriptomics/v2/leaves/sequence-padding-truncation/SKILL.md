---
name: sequence-padding-truncation
description: 'Use when raw single-cell count matrices must be turned into fixed-length
  autoregressive token sequences for tGPT: rank genes by descending expression, tokenize,
  and pad/truncate to 64 or 128 tokens.'
when_to_use_negative:
- Input is already a feature/embedding table or tokenized sequences — re-ranking and
  re-tokenizing would discard or corrupt existing representations.
- 'You need actual expression magnitudes or fold changes: tGPT uses only gene expression
  rankings, so ''the fold changes among genes are neglected and this can affect biological
  interpretation.'''
- 'Analysis targets low-expressing genes: ''tGPT uses only the top-expressing genes;
  therefore, it may miss the information that is specifically represented within the
  low-expressing genes.'''
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0091
tools:
- name: Transformers (PreTrainedTokenizerFast)
  role: Tokenizes ranked gene-symbol lines with truncation and padding to max_len,
    adding <s>/<e> special tokens via the tGPT fast tokenizer.
  repo: https://github.com/deeplearningplus/tGPT
- name: tGPT
  role: Source of the gene vocabulary and tokenizer checkpoint (lixiangchun/transcriptome-gpt-1024-8-16-64)
    over 20,706 protein-coding genes.
  repo: https://github.com/deeplearningplus/tGPT
- name: Scanpy
  role: Loads the single-cell count matrix (e.g., .h5ad / GSE134355) and supplies
    gene symbols used to build descending expression rankings; tested with scanpy==1.9.1.
  repo: https://github.com/theislab/scanpy
- name: pandas
  role: Handles the count matrix and per-cell expression vectors prior to ranking;
    listed as a required package in the README.
  repo: https://github.com/deeplearningplus/tGPT
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.isci.2023.106536
    title: Generative pretraining from large-scale transcriptomes for single-cell
      deciphering
  source_content_hash: sha256:2054c5ec27f3504e085c844b0d1e50172a624420cd6567f13945b85695de5772
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
  - Scanpy
  - Transformers
  - Transformers (PreTrainedTokenizerFast)
  - pandas
  - tGPT
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# sequence-padding-truncation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Converts per-cell ranked gene-symbol sequences into fixed-length token sequences for tGPT by replacing '-'/'.' with underscores, wrapping with [<s>, G1, ..., <e>] special tokens, and padding/truncating to a max_len of 64 (62 genes) or 128 (126 genes) with HuggingFace PreTrainedTokenizerFast.

## When to use

Use when raw single-cell count matrices must be turned into fixed-length autoregressive token sequences for tGPT: rank genes by descending expression, tokenize, and pad/truncate to 64 or 128 tokens.

## When NOT to use

- Input is already a feature/embedding table or tokenized sequences — re-ranking and re-tokenizing would discard or corrupt existing representations.
- You need actual expression magnitudes or fold changes: tGPT uses only gene expression rankings, so 'the fold changes among genes are neglected and this can affect biological interpretation.'
- Analysis targets low-expressing genes: 'tGPT uses only the top-expressing genes; therefore, it may miss the information that is specifically represented within the low-expressing genes.'

## Inputs

- Per-cell gene symbols ranked by descending expression (one line per cell, e.g., ./data/Muris_gene_rankings.txt.gz, gzip-compressed text)
- Single-cell count matrix (e.g., HCL, GEO GSE134355) loaded with Scanpy/pandas
- tGPT HuggingFace tokenizer: lixiangchun/transcriptome-gpt-1024-8-16-64
- max_len parameter: 64 (62 genes) or 128 (126 genes)

## Outputs

- Fixed-length token-ID tensors (attention_mask, input_ids) of length 64 or 128 wrapped as [<s>, G1, ..., <e>], one per cell
- tokenized_ranking_sequences file with one tokenized sequence per cell, all lengths within {64, 128}

## How to apply

Load the count matrix (e.g., HCL or GEO GSE134355) and discard mitochondrial, ribosomal, and non-protein-coding genes against the 20,706 protein-coding gene reference. For each cell, sort gene symbols by descending expression and retain the top 62 (max_len=64) or top 126 (max_len=128) expressed genes — clustering metrics from 62 genes are comparable to 126 per the article. Apply the regex r'\-|\.' substitution of '_' to gene symbols, then tokenize with the tGPT HuggingFace tokenizer (lixiangchun/transcriptome-gpt-1024-8-16-64) calling tokenizer(a, max_length=max_len, truncation=True, padding=True, return_tensors='pt'), which wraps each sequence as [<s>, G1, G2, ..., <e>]. Rationale: rank-based inputs are insensitive to batch effects and normalization, so expression values themselves are discarded before tokenization. Keep batch_size=64 in the DataLoader, and record eos positions via batch.attention_mask.sum(dim=1)-1 so downstream mean-pooling excludes <s> and <e> tokens.

## Related tools

- **Transformers (PreTrainedTokenizerFast)** (Tokenizes ranked gene-symbol lines with truncation and padding to max_len, adding <s>/<e> special tokens via the tGPT fast tokenizer.) — https://github.com/deeplearningplus/tGPT
- **tGPT** (Source of the gene vocabulary and tokenizer checkpoint (lixiangchun/transcriptome-gpt-1024-8-16-64) over 20,706 protein-coding genes.) — https://github.com/deeplearningplus/tGPT
- **Scanpy** (Loads the single-cell count matrix (e.g., .h5ad / GSE134355) and supplies gene symbols used to build descending expression rankings; tested with scanpy==1.9.1.) — https://github.com/theislab/scanpy
- **pandas** (Handles the count matrix and per-cell expression vectors prior to ranking; listed as a required package in the README.) — https://github.com/deeplearningplus/tGPT

## Examples

```
batch = tokenizer(a, max_length=64, truncation=True, padding=True, return_tensors="pt")
```

## Evaluation signals

- All output tokenized sequence lengths fall within {64, 128} matching the chosen max_len (62 or 126 genes plus <s>/<e> special tokens).
- No gene symbol containing '-' or '.' remains in tokenized input — regex r'\-|\.' substitution to '_' is complete for every line.
- attention_mask.sum(dim=1)-1 yields eos indices equal to the number of ranked genes plus one, confirming correct padding of shorter cells.
- Token IDs map into the 20,706 protein-coding gene dictionary with no out-of-vocabulary genes for the retained top-expressing genes.
- Downstream Leiden/UMAP clustering of mean-pooled features (neighbors=20, resolution=0.6) aligns with known cell-type labels, e.g., on Tabula Muris.

## Limitations

- tGPT uses only the top-expressing genes, so information specifically represented in low-expressing genes is missed.
- tGPT uses gene expression rankings but not actual expression levels, so fold changes among genes are neglected, which can affect biological interpretation.
- Clustering metrics from 62 top-expressing genes are comparable to 126, but longer sequences (max_len=128) carry only marginal gains at higher compute cost.

## Evidence
