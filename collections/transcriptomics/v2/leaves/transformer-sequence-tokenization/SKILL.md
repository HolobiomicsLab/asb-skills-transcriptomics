---
name: transformer-sequence-tokenization
description: Use when raw single-cell count matrices (e.g., HCL, GSE134355) must be
  converted into autoregressive transformer input sequences, and gene expression rankings
  — not raw expression values — are the desired model input.
when_to_use_negative:
- When the analysis needs actual expression magnitudes or fold changes among genes
  — tGPT input uses rankings only, so 'fold changes among genes are neglected and
  this can affect biological interpretation'
- When information specifically carried by low-expressing genes matters, since 'tGPT
  uses only the top-expressing genes; therefore, it may miss the information that
  is specifically represented within the low-expressing genes'
- When the input is already a feature/embedding table downstream of tokenization —
  re-tokenizing would be redundant
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3474
tools:
- name: tGPT
  role: Autoregressive transformer model consuming the tokenized ranking sequences;
    provides the GPT2LMHeadModel checkpoint 'lixiangchun/transcriptome-gpt-1024-8-16-64'
  repo: https://github.com/deeplearningplus/tGPT
- name: Transformers
  role: PreTrainedTokenizerFast for tokenizing gene-symbol sequences and GPT2LMHeadModel/GPT2Model
    for loading the pretrained checkpoint
- name: PyTorch
  role: Dataset/DataLoader batching and tensor movement to CUDA for model input
- name: Scanpy
  role: Loading single-cell count matrices (e.g., HCL, GSE134355) and downstream AnnData
    handling of extracted features
  repo: https://github.com/theislab/scanpy
- name: NumPy
  role: Stacking per-cell feature vectors into the final feature matrix (np.stack)
- name: pandas
  role: Reading count matrices and cell type annotation files
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.isci.2023.106536
    title: Generative pretraining from large-scale transcriptomes for single-cell
      deciphering
  source_content_hash: sha256:284f34fe8e2d8bcfe5bae4dcbf5579c32abddb6b2fc52b4ad561f907800851e3
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
  - pandas
  - tGPT
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# transformer-sequence-tokenization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Converts per-cell gene-expression rankings into padded token sequences for transformer (tGPT) input: rank gene symbols by descending expression, sanitize special characters, tokenize with the tGPT HuggingFace tokenizer, and wrap with [<s>, G1, ..., <e>] special tokens to lengths of 64 or 128.

## When to use

Use when raw single-cell count matrices (e.g., HCL, GSE134355) must be converted into autoregressive transformer input sequences, and gene expression rankings — not raw expression values — are the desired model input.

## When NOT to use

- When the analysis needs actual expression magnitudes or fold changes among genes — tGPT input uses rankings only, so 'fold changes among genes are neglected and this can affect biological interpretation'
- When information specifically carried by low-expressing genes matters, since 'tGPT uses only the top-expressing genes; therefore, it may miss the information that is specifically represented within the low-expressing genes'
- When the input is already a feature/embedding table downstream of tokenization — re-tokenizing would be redundant

## Inputs

- Single-cell count matrix (e.g., HCL, GEO GSE134355) loadable with Scanpy/pandas
- 20,706 protein-coding gene reference list
- tGPT HuggingFace tokenizer 'lixiangchun/transcriptome-gpt-1024-8-16-64'
- Gene symbols ranked by expression (e.g., ./data/Muris_gene_rankings.txt.gz text file, one ranked sequence per line)

## Outputs

- Tokenized ranking sequences per cell (tokenized_ranking_sequences.l), one [<s>, G1, ..., <e>] sequence per cell
- Padded/truncated PyTorch tensors (input_ids, attention_mask) of length 64 or 128
- Last_hidden_state feature embeddings extracted from the tGPT transformer (shape n_cells x 1024)

## How to apply

Load the single-cell count matrix (e.g., HCL or GEO GSE134355) with Scanpy/pandas and discard mitochondrial, ribosomal, and non-protein-coding genes against the 20,706 protein-coding gene reference list. For each cell, sort gene symbols by descending expression and retain the top 62 (or 126) expressed genes; rank-based input is deliberately chosen over actual expression levels because it is insensitive to batch effects and normalization. Replace regex-matched '-' and '.' characters in gene symbols with underscores, then map symbols to vocabulary IDs using the PreTrainedTokenizerFast from 'lixiangchun/transcriptome-gpt-1024-8-16-64'. Wrap each ranked sequence as [<s>, G1, G2, ..., <e>] and pad/truncate to max_len of 64 (or 128) via tokenizer(max_length=max_len, truncation=True, padding=True, return_tensors='pt'). Write one tokenized sequence per cell to a tokenized_ranking_sequences.l file and verify all sequence lengths fall within {64, 128}.

## Related tools

- **tGPT** (Autoregressive transformer model consuming the tokenized ranking sequences; provides the GPT2LMHeadModel checkpoint 'lixiangchun/transcriptome-gpt-1024-8-16-64') — https://github.com/deeplearningplus/tGPT
- **Transformers** (PreTrainedTokenizerFast for tokenizing gene-symbol sequences and GPT2LMHeadModel/GPT2Model for loading the pretrained checkpoint)
- **PyTorch** (Dataset/DataLoader batching and tensor movement to CUDA for model input)
- **Scanpy** (Loading single-cell count matrices (e.g., HCL, GSE134355) and downstream AnnData handling of extracted features) — https://github.com/theislab/scanpy
- **NumPy** (Stacking per-cell feature vectors into the final feature matrix (np.stack))
- **pandas** (Reading count matrices and cell type annotation files)

## Examples

```
tokenizer = PreTrainedTokenizerFast.from_pretrained("lixiangchun/transcriptome-gpt-1024-8-16-64"); batch = tokenizer(lines, max_length=64, truncation=True, padding=True, return_tensors="pt")
```

## Evaluation signals

- All output sequences have length exactly 64 or 128 (max_len {64, 128}) after padding/truncation
- Every tokenized sequence starts with <s> and ends with <e> (or is truncated at max_len)
- Gene symbols contain no '-' or '.' characters after regex substitution (replaced with '_')
- All gene tokens map into the 20,706 protein-coding gene dictionary of the tGPT tokenizer
- Downstream clustering metrics from 62 top-expressing genes are comparable to 126 genes (Figure S4), and derived clusters align with known cell labels

## Limitations

- tGPT uses only the top-expressing genes; information represented within low-expressing genes may be missed
- Using gene expression rankings rather than actual levels neglects fold changes among genes, which can affect biological interpretation
- The language modeling objective predicts gene rankings, which is not directly related to biological issues; last-layer representations are not guaranteed to give better clustering

## Evidence
