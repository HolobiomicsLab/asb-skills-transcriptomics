---
name: transformer-decoder-architecture-implementation
description: Use when reproducing or adapting tGPT and you need the exact decoder
  stack (8 layers, 1024 hidden, 16 heads) and config hyperparameters for autoregressive
  pretraining on ranked gene sequences.
when_to_use_negative:
- Input is raw expression counts rather than expression rankings — tGPT models gene-expression
  rankings, not actual expression levels
- The goal requires capturing signal in low-expressing genes — tGPT uses only top-expressing
  genes and may miss low-expression information
- Input is already a feature/embedding table — this skill builds and runs the transformer
  model itself, not downstream clustering
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0203
tools:
- name: PyTorch
  role: torch.nn model construction and torch.no_grad() forward passes for the decoder
    stack
  repo: https://github.com/deeplearningplus/tGPT
- name: Transformers
  role: Provides GPT2LMHeadModel/GPT2Model conventions and PreTrainedTokenizerFast
    for loading the tGPT decoder checkpoint
  repo: https://github.com/deeplearningplus/tGPT
- name: tGPT
  role: The pretrained 8-layer/1024-hidden/16-head decoder checkpoint (transcriptome-gpt-1024-8-16-64)
    being reconstructed or reused
  repo: https://github.com/deeplearningplus/tGPT
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.isci.2023.106536
    title: Generative pretraining from large-scale transcriptomes for single-cell
      deciphering
  source_content_hash: sha256:d75f60d58ac24d4c5c94f45b58bd66c1307613db73855da7a55720787ea42baf
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
  - PyTorch
  - Transformers
  - tGPT
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# transformer-decoder-architecture-implementation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reconstruct the tGPT autoregressive GPT-2-style decoder architecture (8 transformer decoder blocks, 1024 hidden units, 16 attention heads) in PyTorch/Hugging Face, configured per the reported training hyperparameters, to model gene-expression rankings over the 20,706-gene vocabulary.

## When to use

Use when reproducing or adapting tGPT and you need the exact decoder stack (8 layers, 1024 hidden, 16 heads) and config hyperparameters for autoregressive pretraining on ranked gene sequences.

## When NOT to use

- Input is raw expression counts rather than expression rankings — tGPT models gene-expression rankings, not actual expression levels
- The goal requires capturing signal in low-expressing genes — tGPT uses only top-expressing genes and may miss low-expression information
- Input is already a feature/embedding table — this skill builds and runs the transformer model itself, not downstream clustering

## Inputs

- Preprocessed gene-ranking sequences (top-expressing genes, descending order, special tokens [<s>, ..., <e>], max length 64 or 128)
- 20,706-gene vocabulary (gene symbols ranked by expression, e.g. Muris_gene_rankings.txt.gz)
- Tokenizer file (PreTrainedTokenizerFast, e.g. lixiangchun/transcriptome-gpt-1024-8-16-64)

## Outputs

- GPT-2-compatible decoder model (8 blocks, 1024 hidden units, 16 heads) with autoregressive LM head
- config JSON with training hyperparameters (Adam β1=0.9, β2=0.95, lr=0.003, weight decay=0.01, batch size 64, 100 epochs, cosine schedule)
- Hidden states / last_hidden_state feature embeddings (mean-pooled per cell) for downstream clustering

## How to apply

Define a PyTorch model class using Hugging Face GPT2LMHeadModel/GPT2Model conventions: an embedding layer mapping the 20,706-gene vocabulary to 1024-dim vectors plus position encoding, followed by 8 causal (masked) multi-head self-attention decoder blocks with 1024 hidden units and 16 heads, each with position-wise feed-forward layers, and a softmax LM head that autoregressively predicts the next gene in the ranking. Record the config as JSON with Adam optimizer settings β1=0.9, β2=0.95, learning rate 0.003, weight decay 0.01, batch size 64, and 100 epochs with a cosine learning-rate schedule. Inputs are sequences of top-expressing gene rankings formatted as [<s>, G1, G2, G3, …, <e>], padded/truncated to max length 64 (or 128). For inference on pretrained checkpoints, load 'lixiangchun/transcriptome-gpt-1024-8-16-64' with PreTrainedTokenizerFast and GPT2LMHeadModel, run forward passes under torch.no_grad(), and mean-pool last_hidden_state over expressed-gene positions (indices 1 to eos-1) for feature extraction. Rationale: rank-based input makes the model insensitive to batch effects, and the pretraining objective is next-gene-ranking prediction.

## Related tools

- **PyTorch** (torch.nn model construction and torch.no_grad() forward passes for the decoder stack) — https://github.com/deeplearningplus/tGPT
- **Transformers** (Provides GPT2LMHeadModel/GPT2Model conventions and PreTrainedTokenizerFast for loading the tGPT decoder checkpoint) — https://github.com/deeplearningplus/tGPT
- **tGPT** (The pretrained 8-layer/1024-hidden/16-head decoder checkpoint (transcriptome-gpt-1024-8-16-64) being reconstructed or reused) — https://github.com/deeplearningplus/tGPT

## Examples

```
model = GPT2LMHeadModel.from_pretrained("lixiangchun/transcriptome-gpt-1024-8-16-64", output_hidden_states=True).transformer; tokenizer = PreTrainedTokenizerFast.from_pretrained("lixiangchun/transcriptome-gpt-1024-8-16-64")
```

## Evaluation signals

- Forward pass under torch.no_grad() on a tokenized batch (max_length=64) returns last_hidden_state with expected hidden dimension 1024
- Config JSON fields match reported values: n_layer=8, n_embd/hidden=1024, n_head=16, Adam β1=0.9, β2=0.95, lr=0.003, weight decay=0.01, batch size 64, 100 epochs
- Model vocabulary covers the 20,706-gene set and LM head logits rank the next gene over that vocabulary
- Mean-pooled features over positions 1..eos-1 yield a per-cell embedding matrix that supports Leiden clustering aligned with known cell labels
- Checkpoint name encodes the architecture (transcriptome-gpt-1024-8-16-64): 1024 hidden, 8 layers, 16 heads, max length 64

## Limitations

- tGPT uses only the top-expressing genes; information specifically represented within low-expressing genes may be missed
- tGPT uses gene expression rankings but not actual expression levels, so fold changes among genes are neglected, which can affect biological interpretation
- The language-modeling objective predicts gene rankings, which is not directly related to biological issues; last-layer representations are not guaranteed to give better clustering

## Evidence
