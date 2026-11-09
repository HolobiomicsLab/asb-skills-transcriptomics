---
name: survival-association-analysis
description: Use when you have per-sample bulk-transcriptome embeddings or attention-head
  features from tGPT plus TCGA overall survival annotations, and you need to test
  prognostic significance per cancer type.
when_to_use_negative:
- Input is single-cell data for clustering or trajectory tasks — this skill is specific
  to bulk tumor transcriptomes with survival/genomic-alteration annotations.
- Actual expression levels are required rather than expression rankings — tGPT consumes
  top-expressing gene rankings and neglects fold changes among genes.
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: tGPT
  role: Pretrained transformer (GPT2LMHeadModel) used to embed bulk transcriptomes
    and expose per-head attention distributions for survival association testing
  repo: https://github.com/deeplearningplus/tGPT
- name: PreTrainedTokenizerFast
  role: Tokenizes the ranked top-64 gene symbol string per tumor sample (max_len=64)
    before model inference
  repo: https://github.com/deeplearningplus/tGPT
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.isci.2023.106536
    title: Generative pretraining from large-scale transcriptomes for single-cell
      deciphering
  source_content_hash: sha256:8e26f9cced6e66c121c214b21e601915163e078ca9b2589aeefc62c70d2eeddc
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
  - PreTrainedTokenizerFast
  - tGPT
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# survival-association-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Tests whether tGPT-derived bulk-tumor features (per-attention-head entropy/importance and mean-pooled embeddings) are prognostic for overall survival, as done in the TCGA pan-cancer analysis of tGPT (Figure 5B-5F).

## When to use

Use when you have per-sample bulk-transcriptome embeddings or attention-head features from tGPT plus TCGA overall survival annotations, and you need to test prognostic significance per cancer type.

## When NOT to use

- Input is single-cell data for clustering or trajectory tasks — this skill is specific to bulk tumor transcriptomes with survival/genomic-alteration annotations.
- Actual expression levels are required rather than expression rankings — tGPT consumes top-expressing gene rankings and neglects fold changes among genes.

## Inputs

- TCGA bulk tumor transcriptomes (gene expression rankings per sample, one sample per row of gene symbols)
- Pretrained tGPT checkpoint lixiangchun/transcriptome-gpt-1024-8-16-64
- Per-cancer-type clinical annotations: overall survival status/time, TP53 mutation, HRD, genome doubling, TMB
- Gene symbol list ranked by descending expression per sample (e.g. gzip-compressed ranking text file)

## Outputs

- attention_head_associations.csv: per-head test statistics, p-values, and significant head-cancer/event pairs mirroring Figure 5B-5F
- Per-sample mean-pooled tGPT embeddings and per-head attention entropy/importance features

## How to apply

Load the pretrained tGPT checkpoint (lixiangchun/transcriptome-gpt-1024-8-16-64) as a GPT2LMHeadModel with output_hidden_states=True on GPU in eval mode. Rank genes in each TCGA bulk tumor transcriptome by descending expression, tokenize the top 64 genes (max_len=64, batch size 64) with the PreTrainedTokenizerFast tokenizer, replacing '-' and '.' in gene symbols with '_'. Under torch.no_grad, mean-pool last_hidden_state over gene-token positions 1 through the EOS index (per attention_mask) to obtain per-sample embeddings while recording per-head attention distributions; then compute per-head entropy and importance features. Aggregate features within each of the 33 TCGA cancer types and test associations with overall survival (alongside TP53 mutation, HRD, genome doubling, and TMB) using regression-based testing within each cancer type.

## Related tools

- **tGPT** (Pretrained transformer (GPT2LMHeadModel) used to embed bulk transcriptomes and expose per-head attention distributions for survival association testing) — https://github.com/deeplearningplus/tGPT
- **PreTrainedTokenizerFast** (Tokenizes the ranked top-64 gene symbol string per tumor sample (max_len=64) before model inference) — https://github.com/deeplearningplus/tGPT

## Examples

```
model = GPT2LMHeadModel.from_pretrained('lixiangchun/transcriptome-gpt-1024-8-16-64', output_hidden_states=True).transformer.cuda().eval()
```

## Evaluation signals

- Prognostic significance of attention-head features is reproduced across 7 cancer types (per Figure 5B-5F validation).
- Head-level features also reproduce significant TP53 associations across 9 cancer types, HRD across 4, and genome doubling across 14 — a cross-event sanity check.
- Per-sample embeddings equal the mean of last_hidden_state over token positions 1..EOS index as computed from attention_mask.sum(dim=1)-1.
- attention_head_associations.csv contains test statistics, p-values, and head-cancer/event pairs for all 33 TCGA cancer types.

## Limitations

- tGPT uses only top-expressing genes, so information represented in low-expressing genes may be missed.
- tGPT uses gene expression rankings, not actual expression levels; fold changes among genes are neglected, which can affect biological interpretation.
- The language modeling objective predicts gene rankings and is not directly related to biological issues; further validation is required.

## Evidence
