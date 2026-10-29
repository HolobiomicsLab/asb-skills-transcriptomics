---
name: tcga-pancancer-association-testing
description: Use when testing whether tGPT-derived attention-head features from bulk
  tumor transcriptomes are associated with genomic alteration events, prognosis, or
  survival outcomes in a pan-cancer cohort.
when_to_use_negative:
- Task is single-cell clustering or cell-lineage inference from scRNA-seq — use tGPT's
  embedding + Leiden/UMAP workflow instead of attention-head association testing
- Input is already a feature/embedding table — attention-head distributions must be
  re-recorded from the model, which requires ranked gene-symbol lines as input
- No clinical or molecular event annotations (TP53, HRD, genome doubling, TMB, survival)
  are available — the association regression cannot be run
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0622
tools:
- name: tGPT
  role: Pretrained transformer (checkpoint lixiangchun/transcriptome-gpt-1024-8-16-64)
    run as GPT2LMHeadModel to obtain embeddings and per-head attention distributions
    from ranked bulk transcriptomes
  repo: https://github.com/deeplearningplus/tGPT
- name: PreTrainedTokenizerFast
  role: HuggingFace tokenizer from the tGPT checkpoint used to tokenize top-64 gene
    rankings (max_len=64) with '-' and '.' replaced by '_'
  repo: https://github.com/deeplearningplus/tGPT
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.isci.2023.106536
    title: Generative pretraining from large-scale transcriptomes for single-cell
      deciphering
  source_content_hash: sha256:c1f6583f34e03466c987a43945d4648b40d1f31394fd431a85402a99280d7874
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
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 4
---

# tcga-pancancer-association-testing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reproduce tGPT attention-head association analyses on TCGA bulk tumor transcriptomes: compute per-head attention entropy/importance features and test them against molecular alteration events (TMB, TP53 mutation, HRD, genome doubling) and overall survival across 33 cancer types.

## When to use

Use when testing whether tGPT-derived attention-head features from bulk tumor transcriptomes are associated with genomic alteration events, prognosis, or survival outcomes in a pan-cancer cohort.

## When NOT to use

- Task is single-cell clustering or cell-lineage inference from scRNA-seq — use tGPT's embedding + Leiden/UMAP workflow instead of attention-head association testing
- Input is already a feature/embedding table — attention-head distributions must be re-recorded from the model, which requires ranked gene-symbol lines as input
- No clinical or molecular event annotations (TP53, HRD, genome doubling, TMB, survival) are available — the association regression cannot be run

## Inputs

- TCGA bulk tumor transcriptomes with genes ranked by descending expression (gene-rank text lines)
- Per-sample clinical/genomic annotations: TMB, TP53 mutation status, HRD, genome doubling, overall survival, cancer type (33 TCGA types)

## Outputs

- attention_head_associations.csv with test statistics, p-values, and significant head-cancer/event pairs (Figures 5B-5F)
- Per-sample per-head attention entropy/importance feature tables aggregated per cancer type

## How to apply

Load the pretrained tGPT checkpoint lixiangchun/transcriptome-gpt-1024-8-16-64 as a GPT2LMHeadModel with output_hidden_states=True on GPU in evaluation mode. Rank genes in each TCGA bulk transcriptome by descending expression, tokenize the top 64 genes (max_len=64) with a PreTrainedTokenizerFast in batches of 64, replacing '-' and '.' in gene symbols with '_'. Under torch.no_grad, mean-pool last_hidden_state over gene-token positions 1 through the EOS index (per attention_mask) to obtain per-sample embeddings while recording per-head attention distributions. Compute per-head attention entropy and importance features per sample, aggregate within each of the 33 cancer types, and test associations with annotated events (TP53 mutation, HRD, genome doubling, TMB, overall survival) using regression-based testing within each cancer type. Compile test statistics and p-values into attention_head_associations.csv, mirroring Figure 5B-5F.

## Related tools

- **tGPT** (Pretrained transformer (checkpoint lixiangchun/transcriptome-gpt-1024-8-16-64) run as GPT2LMHeadModel to obtain embeddings and per-head attention distributions from ranked bulk transcriptomes) — https://github.com/deeplearningplus/tGPT
- **PreTrainedTokenizerFast** (HuggingFace tokenizer from the tGPT checkpoint used to tokenize top-64 gene rankings (max_len=64) with '-' and '.' replaced by '_') — https://github.com/deeplearningplus/tGPT

## Examples

```
tokenizer = PreTrainedTokenizerFast.from_pretrained("lixiangchun/transcriptome-gpt-1024-8-16-64"); model = GPT2LMHeadModel.from_pretrained("lixiangchun/transcriptome-gpt-1024-8-16-64", output_hidden_states=True).transformer.to("cuda").eval()
```

## Evaluation signals

- Significant attention-head associations with TP53 mutations reproduced across 9 cancer types at the pan-cancer level
- Significant associations with HRD across 4 cancer types and with genome doubling across 14 cancer types
- Prognostic (overall survival) significance of head features reproduced across 7 cancer types
- Per-sample embedding dimensionality matches mean-pooling over token positions 1..EOS index (attention_mask.sum(dim=1)-1) for every sample
- Output attention_head_associations.csv contains test statistics, p-values, and head-cancer/event pairs for all 33 cancer types

## Limitations

- tGPT uses only the top-expressing genes; information specifically represented within low-expressing genes may be missed
- tGPT uses gene expression rankings but not actual expression levels, so fold changes among genes are neglected, which can affect biological interpretation
- The language modeling objective predicts gene rankings, which is not directly related to biological issues; further study is required

## Evidence
