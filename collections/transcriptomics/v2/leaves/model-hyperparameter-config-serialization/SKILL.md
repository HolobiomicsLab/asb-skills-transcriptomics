---
name: model-hyperparameter-config-serialization
description: 'Use when reimplementing or reproducing tGPT: you need the exact architecture
  dimensions (8 blocks / 1024 hidden / 16 heads) and training hyperparameters written
  into a config before building the PyTorch model.'
when_to_use_negative:
- You only need to run inference or feature extraction with an existing pretrained
  checkpoint — loading via GPT2LMHeadModel.from_pretrained suffices without rebuilding
  the config
- Your task requires actual expression magnitudes rather than gene-expression rankings
  — tGPT's modeling objective predicts rankings and neglects fold changes
- The transformer architecture dimensions differ from the reported tGPT configuration
  (8 blocks / 1024 hidden / 16 heads), e.g. when adapting to a different pretraining
  corpus
tools:
- name: tGPT
  role: Target model whose architecture (8 decoder blocks, 1024 hidden units, 16 attention
    heads) and hyperparameters are serialized into the config
  repo: https://github.com/deeplearningplus/tGPT
- name: PyTorch
  role: torch.nn framework used to define the model class, embedding layer, decoder
    blocks, and run verification forward passes
- name: Transformers
  role: Provides GPT2LMHeadModel/GPT2Model conventions and PreTrainedTokenizerFast
    for building and validating the tGPT architecture
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.isci.2023.106536
    title: Generative pretraining from large-scale transcriptomes for single-cell
      deciphering
  source_content_hash: sha256:e7f0a3735d7061957d0223ac9ce0527c2ea8661b20f17d633a0fc03db14332c5
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
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# model-hyperparameter-config-serialization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reconstructs and serializes the tGPT autoregressive transformer architecture — 8 decoder blocks, 1024 hidden units, 16 attention heads over a 20,706-gene vocabulary — together with a config JSON capturing reported training hyperparameters so the model can be faithfully rebuilt or retrained.

## When to use

Use when reimplementing or reproducing tGPT: you need the exact architecture dimensions (8 blocks / 1024 hidden / 16 heads) and training hyperparameters written into a config before building the PyTorch model.

## When NOT to use

- You only need to run inference or feature extraction with an existing pretrained checkpoint — loading via GPT2LMHeadModel.from_pretrained suffices without rebuilding the config
- Your task requires actual expression magnitudes rather than gene-expression rankings — tGPT's modeling objective predicts rankings and neglects fold changes
- The transformer architecture dimensions differ from the reported tGPT configuration (8 blocks / 1024 hidden / 16 heads), e.g. when adapting to a different pretraining corpus

## Inputs

- tGPT architecture specification from the article (8 decoder blocks, 1024 hidden units, 16 attention heads)
- Reported training hyperparameters (Adam beta1=0.9, beta2=0.95, lr 0.003, weight decay 0.01, batch size 64, 100 epochs, cosine schedule)
- Hugging Face transformers checkpoint name (e.g. lixiangchun/transcriptome-gpt-1024-8-16-64)
- Gene-ranking sequences from task_001 (top-expressing gene rankings, max length 64)

## Outputs

- PyTorch (torch.nn) model class implementing the tGPT autoregressive decoder with 20,706-gene vocabulary
- Config JSON recording architecture and training hyperparameters
- Hidden states from a verification forward pass of expected shape

## How to apply

Define a torch.nn model class using GPT2LMHeadModel/GPT2Model conventions from Hugging Face transformers, with an embedding layer mapping gene tokens from the 20,706-gene vocabulary to 1024-dim vectors plus position encoding, then stack 8 causal multi-head decoder blocks with 1024 hidden units and 16 heads, each followed by position-wise feed-forward layers, and attach a softmax language-model head predicting next-gene rankings. Write a config JSON recording the reported hyperparameters: Adam with beta1=0.9, beta2=0.95, learning rate 0.003, weight decay 0.01, batch size 64, 100 epochs, and a cosine learning-rate schedule. The checkpoint name 'transcriptome-gpt-1024-8-16-64' encodes these dimensions and can be used to cross-check the config. Verify by loading preprocessed gene-ranking sequences (max length 64), running a forward pass under torch.no_grad(), and confirming hidden-state shapes match the architecture.

## Related tools

- **tGPT** (Target model whose architecture (8 decoder blocks, 1024 hidden units, 16 attention heads) and hyperparameters are serialized into the config) — https://github.com/deeplearningplus/tGPT
- **PyTorch** (torch.nn framework used to define the model class, embedding layer, decoder blocks, and run verification forward passes)
- **Transformers** (Provides GPT2LMHeadModel/GPT2Model conventions and PreTrainedTokenizerFast for building and validating the tGPT architecture)

## Examples

```
from transformers import PreTrainedTokenizerFast, GPT2LMHeadModel; model = GPT2LMHeadModel.from_pretrained("lixiangchun/transcriptome-gpt-1024-8-16-64", output_hidden_states=True).transformer
```

## Evaluation signals

- Config JSON fields exactly match reported values: Adam beta1=0.9, beta2=0.95, lr 0.003, weight decay 0.01, batch size 64, 100 epochs, cosine learning-rate schedule
- Architecture dimensions in config match checkpoint identifier 'transcriptome-gpt-1024-8-16-64' (1024 hidden, 8 layers, 16 heads, max_len 64)
- Forward pass under torch.no_grad() returns hidden states of expected shape (batch_size x max_len x 1024) with no runtime errors
- Model head outputs next-gene predictions over the 20,706-gene vocabulary sized tensor
- Tokenizer (PreTrainedTokenizerFast) loads the same vocabulary as the checkpoint and truncation/padding to max_length=64 behaves as in the README extract-features code

## Limitations

- Pretrained tGPT weights/checkpoint availability for reuse is not stated beyond the code repository link and Hugging Face checkpoint name
- Feature representations from the last transformer layer are not guaranteed to give better clustering, so config serialization should also support extracting embedding-layer and intermediate-layer outputs
- tGPT uses only top-expressing genes and gene rankings (not actual expression levels), which constrains what the serialized model can represent

## Evidence
