---
name: pytorch-forward-pass-verification
description: Use when reproducing tGPT's architecture from the paper's reported hyperparameters
  and you must confirm the model runs end-to-end on rank-encoded gene sequences before
  pretraining or feature extraction.
when_to_use_negative:
- The goal is biological interpretation of expression fold changes — tGPT consumes
  gene expression rankings, not actual expression levels, so fold changes among genes
  are neglected.
- Only low-expressing genes carry the signal of interest — tGPT uses only top-expressing
  genes and may miss information represented within the low-expressing genes.
- Input is already a cell-level feature/embedding table — verification of the transformer
  forward pass requires raw tokenized gene-ranking sequences.
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: tGPT
  role: the reconstructed autoregressive transformer architecture (8 decoder blocks,
    1024 hidden units, 16 attention heads) whose forward pass is verified
  repo: github.com/deeplearningplus/tGPT
- name: PyTorch
  role: torch.nn model definition, torch.no_grad() forward pass, and shape assertions
    on hidden states
- name: Transformers
  role: provides PreTrainedTokenizerFast and GPT2LMHeadModel/GPT2Model conventions
    used to build and load the tGPT model and tokenize gene-ranking sequences
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.isci.2023.106536
    title: Generative pretraining from large-scale transcriptomes for single-cell
      deciphering
  source_content_hash: sha256:4972738bc30d886b7933eed803fb241a0266c7f85642f0bdb3ee6ee42b290b84
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/deeplearningplus/tGPT
  tools:
  - PyTorch
  - Transformers
  - tGPT
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# pytorch-forward-pass-verification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reconstructs the tGPT autoregressive transformer (8 decoder blocks, 1024 hidden units, 16 attention heads) in PyTorch and verifies a forward pass over preprocessed gene-ranking sequences produces correctly shaped hidden states matching the reported configuration.

## When to use

Use when reproducing tGPT's architecture from the paper's reported hyperparameters and you must confirm the model runs end-to-end on rank-encoded gene sequences before pretraining or feature extraction.

## When NOT to use

- The goal is biological interpretation of expression fold changes — tGPT consumes gene expression rankings, not actual expression levels, so fold changes among genes are neglected.
- Only low-expressing genes carry the signal of interest — tGPT uses only top-expressing genes and may miss information represented within the low-expressing genes.
- Input is already a cell-level feature/embedding table — verification of the transformer forward pass requires raw tokenized gene-ranking sequences.

## Inputs

- preprocessed gene-ranking sequences (top-expressing gene rankings, max length 64) from the 20,706-gene vocabulary, e.g. Muris_gene_rankings.txt.gz
- PreTrainedTokenizerFast tokenizer file (e.g. lixiangchun/transcriptome-gpt-1024-8-16-64)
- PyTorch model definition / checkpoint configuration (GPT2LMHeadModel-style)

## Outputs

- instantiated tGPT model (8 decoder blocks, 1024 hidden units, 16 attention heads)
- hidden states / last_hidden_state tensors of shape (batch, max_len, 1024) from a verified forward pass
- config JSON recording training hyperparameters (Adam beta1=0.9, beta2=0.95, lr 0.003, weight decay 0.01, batch size 64, 100 epochs, cosine schedule)

## How to apply

Define a PyTorch model (following the GPT2LMHeadModel/GPT2Model conventions imported from Hugging Face transformers in the tGPT README) whose embedding maps the 20,706-gene vocabulary to 1024-dim vectors, stack 8 causal (masked) multi-head self-attention decoder blocks with 16 heads, and attach a softmax language-model head that autoregressively predicts next-gene rankings. Load preprocessed gene-ranking sequences (top-expressing gene rankings, max length 64, tokenized with PreTrainedTokenizerFast) and run a forward pass under torch.no_grad() with output_hidden_states=True, checking that last_hidden_state has the expected shape (batch, <=64, 1024). Write a config JSON recording the reported training hyperparameters: Adam with beta1=0.9, beta2=0.95, learning rate 0.003, weight decay 0.01, batch size 64, 100 epochs with cosine learning-rate schedule, and assert all config fields match the reported values. Mean-pool last_hidden_state over non-padding positions (up to each sequence's eos index, as in the README extract-features loop) to sanity-check that features downstream of the transformer are well-formed.

## Related tools

- **tGPT** (the reconstructed autoregressive transformer architecture (8 decoder blocks, 1024 hidden units, 16 attention heads) whose forward pass is verified) — github.com/deeplearningplus/tGPT
- **PyTorch** (torch.nn model definition, torch.no_grad() forward pass, and shape assertions on hidden states)
- **Transformers** (provides PreTrainedTokenizerFast and GPT2LMHeadModel/GPT2Model conventions used to build and load the tGPT model and tokenize gene-ranking sequences)

## Examples

```
model = GPT2LMHeadModel.from_pretrained(checkpoint, output_hidden_states=True).transformer.to(device); batch = tokenizer(lines, max_length=64, truncation=True, padding=True, return_tensors='pt'); x = model(**{k: v.to(device) for k, v in batch.items()}); assert x.last_hidden_state.shape[-1] == 1024
```

## Evaluation signals

- Forward pass under torch.no_grad() with output_hidden_states=True returns last_hidden_state of shape (batch, <=max_len, 1024) where max_len=64.
- Model exposes exactly 8 transformer decoder layers and 16 attention heads with 1024-dim hidden units, matching the reported architecture.
- Embedding layer vocabulary size equals 20,706 gene tokens plus special tokens.
- All fields in the written config JSON match reported values: Adam beta1=0.9, beta2=0.95, lr 0.003, weight decay 0.01, batch size 64, 100 epochs, cosine schedule.
- Mean-pooled features per cell (averaging positions 1..eos_idx-1 of last_hidden_state) have dimension 1024 and are finite (no NaNs).

## Limitations

- Pretrained tGPT weights/checkpoint availability for reuse is not stated beyond the code repository link (github.com/deeplearningplus/tGPT).
- The language modeling objective predicts gene rankings, which is not directly related to biological issues; passing verification does not guarantee biologically meaningful representations.
- Feature representation from the last transformer layer is not guaranteed to give better clustering, since the pretraining objective is not directly related to cell clustering.

## Evidence

- [other] tGPT's described network architecture consists of 8 transformer decoder blocks with 1024 hidden units and 16 attention heads.: [evidence span withheld — anchor status: not_found]
