---
name: genomic-alteration-significance-testing
description: Use when you have per-sample attention-head features from a pretrained
  tGPT model over tumor bulk transcriptomes and need to link them to mutation, HRD,
  genome-doubling, TMB, or survival annotations.
when_to_use_negative:
- Input is single-cell data for clustering or trajectory tasks — tGPT attention-head
  alteration testing targets bulk tumor tissue feature patterns
- You lack per-sample genomic alteration or survival annotations — the regression
  tests cannot be run on embeddings alone
- Only actual expression levels (not gene expression rankings) are available and cannot
  be rank-transformed — tGPT input requires ranked top-expressing genes
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3517
tools:
- name: tGPT
  role: Pretrained transformer (checkpoint lixiangchun/transcriptome-gpt-1024-8-16-64)
    whose per-head attention distributions provide the entropy/importance features
    tested against genomic alteration events
  repo: github.com/deeplearningplus/tGPT
- name: transformers (GPT2LMHeadModel, PreTrainedTokenizerFast)
  role: Loads the tGPT checkpoint with output_hidden_states=True and tokenizes ranked
    top-64 gene sequences (max_len=64) for batch inference
- name: torch
  role: GPU inference of the tGPT transformer under torch.no_grad and computation
    of EOS-index mean pooling of last_hidden_state
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.isci.2023.106536
    title: Generative pretraining from large-scale transcriptomes for single-cell
      deciphering
  source_content_hash: sha256:e1c5c400d2adc03fb1089ccc937a177604f37944205de7b9db3a71e2f6664fd1
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/deeplearningplus/tGPT
  tools:
  - tGPT
  - torch
  - transformers (GPT2LMHeadModel, PreTrainedTokenizerFast)
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3517
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# genomic-alteration-significance-testing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Tests whether tGPT attention-head entropy/importance features computed from tumor bulk transcriptomes are significantly associated with genomic alteration events (TMB, TP53 mutation, HRD, genome doubling) and overall survival, using regression-based testing per cancer type.

## When to use

Use when you have per-sample attention-head features from a pretrained tGPT model over tumor bulk transcriptomes and need to link them to mutation, HRD, genome-doubling, TMB, or survival annotations.

## When NOT to use

- Input is single-cell data for clustering or trajectory tasks — tGPT attention-head alteration testing targets bulk tumor tissue feature patterns
- You lack per-sample genomic alteration or survival annotations — the regression tests cannot be run on embeddings alone
- Only actual expression levels (not gene expression rankings) are available and cannot be rank-transformed — tGPT input requires ranked top-expressing genes

## Inputs

- TCGA bulk tumor transcriptomes (gene expression rankings per sample)
- Pretrained tGPT checkpoint 'lixiangchun/transcriptome-gpt-1024-8-16-64'
- Per-sample molecular annotations: TP53 mutation status, TMB, HRD, genome doubling
- TCGA overall survival annotations and cancer-type labels (33 types)

## Outputs

- Per-sample tGPT embeddings and per-attention-head entropy/importance features
- attention_head_associations.csv with test statistics, p-values, and significant head-cancer/event pairs
- Significant association counts mirroring Figure 5B-5F (TP53 across 9 cancer types, HRD across 4, genome doubling across 14, prognostic significance across 7)

## How to apply

Load the pretrained tGPT checkpoint lixiangchun/transcriptome-gpt-1024-8-16-64 as a GPT2LMHeadModel with output_hidden_states=True on GPU and set it to evaluation mode. Rank genes in each bulk tumor transcriptome by descending expression, tokenize the top 64 genes (max_len=64) with PreTrainedTokenizerFast in batches of 64, replacing '-' and '.' in gene symbols with '_'. Run the transformer under torch.no_grad, mean-pool last_hidden_state over gene-token positions 1 through the EOS index (derived from attention_mask), and record per-head attention distributions. Compute per-head entropy and importance features per sample, aggregate within each of the 33 TCGA cancer types, and test associations with TP53 mutation, HRD, genome doubling, TMB, and overall survival using regression-based testing within each cancer type. Compile all test statistics, p-values, and significant head-cancer/event pairs into a single CSV table.

## Related tools

- **tGPT** (Pretrained transformer (checkpoint lixiangchun/transcriptome-gpt-1024-8-16-64) whose per-head attention distributions provide the entropy/importance features tested against genomic alteration events) — github.com/deeplearningplus/tGPT
- **transformers (GPT2LMHeadModel, PreTrainedTokenizerFast)** (Loads the tGPT checkpoint with output_hidden_states=True and tokenizes ranked top-64 gene sequences (max_len=64) for batch inference)
- **torch** (GPU inference of the tGPT transformer under torch.no_grad and computation of EOS-index mean pooling of last_hidden_state)

## Examples

```
model = GPT2LMHeadModel.from_pretrained('lixiangchun/transcriptome-gpt-1024-8-16-64', output_hidden_states=True).transformer.to(device); model.eval()
```

## Evaluation signals

- Reproduces significant attention-head associations with TP53 mutations across 9 cancer types at the pan-cancer and cancer-type level
- Reproduces HRD associations across 4 cancer types and genome-doubling associations across 14 cancer types
- Reproduces prognostic (overall survival) significance of attention-head features across 7 cancer types
- Output attention_head_associations.csv contains test statistics and p-values for all head-cancer/event pairs mirroring Figure 5B-5F
- Tokenization invariant: '-' and '.' replaced with '_' in gene symbols; embeddings mean-pooled over positions 1 through EOS index per attention_mask

## Limitations

- tGPT uses only the top-expressing genes; therefore, it may miss information specifically represented within the low-expressing genes
- tGPT uses gene expression rankings but not actual expression levels, so fold changes among genes are neglected and this can affect biological interpretation
- The language modeling objective predicts gene rankings, which is not directly related to biological issues; associations require further study

## Evidence
