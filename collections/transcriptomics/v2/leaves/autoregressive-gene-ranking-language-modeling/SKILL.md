---
name: autoregressive-gene-ranking-language-modeling
description: Use when building a transformer language model whose autoregressive pretraining
  target is gene expression rankings (not counts) of single-cell transcriptomes, e.g.
  reproducing tGPT or adapting GPT2LMHeadModel to a gene-token vocabulary.
when_to_use_negative:
- The goal is modeling actual expression levels or fold changes — tGPT input is gene
  expression rankings only, so magnitude information is lost
- The analysis depends on information from low-expressing genes — tGPT uses only top-expressing
  genes (62 or 126) and may miss signals in low-expressing genes
- Only raw, unfiltered counts are available — cells must first be filtered (expressed
  genes <200 or mitochondrial counts >30% removed) and mitochondrial/ribosomal/non-protein-coding
  genes discarded before rank construction
edam_operation: http://edamontology.org/operation_0242
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_3308
tools:
- name: tGPT
  role: The transformer-based generative pretrained model being reconstructed; autoregressively
    models gene ranking in the context of preceding neighbors
  repo: https://github.com/deeplearningplus/tGPT
- name: PyTorch
  role: Framework (torch==1.7.1, python==3.7.11) in which the embedding, 8 decoder
    blocks, and forward pass under torch.no_grad() are implemented
- name: Transformers
  role: Hugging Face library supplying PreTrainedTokenizerFast tokenization and GPT2LMHeadModel/GPT2Model
    conventions for the tGPT architecture and checkpoint loading
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.isci.2023.106536
    title: Generative pretraining from large-scale transcriptomes for single-cell
      deciphering
  source_content_hash: sha256:60878b0a418313d18ed61b8c8d21eaf04c2929a854860415b0bf206b6f586ae3
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
  edam_operation: http://edamontology.org/operation_0242
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# autoregressive-gene-ranking-language-modeling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reconstruct and pretrain tGPT, a GPT-style autoregressive transformer (8 decoder blocks, 1024 hidden units, 16 attention heads) that generatively models the rankings of top-expressing genes per cell over a 20,706-gene vocabulary, enabling integration of 22.3 million single-cell transcriptomes insensitive to batch effects.

## When to use

Use when building a transformer language model whose autoregressive pretraining target is gene expression rankings (not counts) of single-cell transcriptomes, e.g. reproducing tGPT or adapting GPT2LMHeadModel to a gene-token vocabulary.

## When NOT to use

- The goal is modeling actual expression levels or fold changes — tGPT input is gene expression rankings only, so magnitude information is lost
- The analysis depends on information from low-expressing genes — tGPT uses only top-expressing genes (62 or 126) and may miss signals in low-expressing genes
- Only raw, unfiltered counts are available — cells must first be filtered (expressed genes <200 or mitochondrial counts >30% removed) and mitochondrial/ribosomal/non-protein-coding genes discarded before rank construction

## Inputs

- Gene-ranking text file of top-expressing gene symbols per cell (e.g. ./data/Muris_gene_rankings.txt.gz, ranked by descending expression)
- 20,706-gene token vocabulary / fast tokenizer file (lixiangchun/transcriptome-gpt-1024-8-16-64)
- Preprocessed single-cell transcriptomes (mitochondrial/ribosomal/non-protein-coding genes discarded; cells with <200 expressed genes or >30% mito counts filtered)

## Outputs

- GPT2LMHeadModel checkpoint configured as 8 decoder blocks, 1024 hidden units, 16 attention heads, max sequence length 64
- config JSON with reported hyperparameters (Adam beta1=0.9 beta2=0.95, lr 0.003, weight decay 0.01, batch size 64, 100 epochs, cosine schedule)
- Hidden states / last_hidden_state embeddings per cell from the embedding and 8 transformer layers, used for KNN-Leiden clustering

## How to apply

Define a PyTorch/Hugging Face GPT2LMHeadModel-based model with an embedding layer mapping the 20,706-gene token vocabulary to 1024-dim vectors, stacked with 8 causal (masked) transformer decoder blocks using 1024 hidden units and 16 attention heads, and a softmax LM head that autoregressively predicts the next gene in the ranking. Formulate each cell as the sequence [<s>, G1, G2, G3, ..., <e>] of descending-sorted top-expressing gene symbols (62 or 126 genes), padded/truncated to max length 64 or 128 with PreTrainedTokenizerFast. Pretrain with Adam (beta1=0.9, beta2=0.95), learning rate 0.003, weight decay 0.01, batch size 64, for 100 epochs under a cosine learning-rate schedule. Rationale: rank-based inputs make the model insensitive to batch effects and normalization, allowing integration of unlimited numbers of cells across platforms. Verify by a torch.no_grad() forward pass on a batch of gene-ranking sequences confirming hidden-state shapes match the 8-layer/1024-dim/16-head configuration.

## Related tools

- **tGPT** (The transformer-based generative pretrained model being reconstructed; autoregressively models gene ranking in the context of preceding neighbors) — https://github.com/deeplearningplus/tGPT
- **PyTorch** (Framework (torch==1.7.1, python==3.7.11) in which the embedding, 8 decoder blocks, and forward pass under torch.no_grad() are implemented)
- **Transformers** (Hugging Face library supplying PreTrainedTokenizerFast tokenization and GPT2LMHeadModel/GPT2Model conventions for the tGPT architecture and checkpoint loading)

## Examples

```
model = GPT2LMHeadModel.from_pretrained("lixiangchun/transcriptome-gpt-1024-8-16-64", output_hidden_states=True).transformer
```

## Evaluation signals

- Hidden states from a torch.no_grad() forward pass have the expected shape for 8 transformer layers with 1024 hidden units and 16 attention heads
- All config fields match reported values: Adam beta1=0.9, beta2=0.95, learning rate 0.003, weight decay 0.01, batch size 64, 100 epochs, cosine schedule
- Tokenizer vocabulary maps the 20,706-gene space and sequences are padded/truncated to max length 64 (or 128)
- Extracted last_hidden_state features (mean-pooled over gene tokens) yield Leiden clusters and trajectories aligned with known cell type labels after sc.pp.neighbors(n_neighbors=20)
- Feature representations extracted from the embedding layer and all 8 transformer layers can each support KNN graph clustering with Leiden

## Limitations

- tGPT uses only the top-expressing genes, so information specifically represented within low-expressing genes may be missed
- tGPT uses gene expression rankings but not actual expression levels; fold changes among genes are neglected, which can affect biological interpretation
- The language modeling objective predicts gene rankings, which is not directly related to biological issues, and the last transformer layer representation is not guaranteed to give better clustering
- 62 vs 126 top-expressing genes give comparable clustering metrics, so longer input sequences add limited value

## Evidence
