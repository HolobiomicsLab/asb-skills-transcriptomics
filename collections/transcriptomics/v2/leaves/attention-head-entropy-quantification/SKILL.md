---
name: attention-head-entropy-quantification
description: Use when bulk tumor transcriptomes must be converted to gene-expression
  rankings and passed through pretrained tGPT (checkpoint lixiangchun/transcriptome-gpt-1024-8-16-64)
  with output_attentions=True to extract per-head entropy/importance features for
  clinical or molecular association testing.
when_to_use_negative:
- Input is already a per-sample feature/embedding table — this skill requires raw
  ranked gene-expression sequences to pass through the transformer and collect attention
  matrices
- Single-cell analysis tasks (clustering, batch integration, trajectory inference)
  where the standard tGPT feature-extraction plus Leiden/UMAP workflow is the appropriate
  route rather than attention-head statistics
- Data cannot be tokenized into the 64-token ranked gene format expected by the 'lixiangchun/transcriptome-gpt-1024-8-16-64'
  checkpoint (e.g. non-transcriptomic measurements)
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: tGPT
  role: Pretrained transformer (GPT2LMHeadModel/GPT2Model checkpoint lixiangchun/transcriptome-gpt-1024-8-16-64)
    run with output_attentions=True to supply the 8-layer, per-head attention distributions
    from ranked bulk transcriptomes
  repo: https://github.com/deeplearningplus/tGPT
- name: Transformers
  role: Provides PreTrainedTokenizerFast for tokenizing the 64-gene ranking sequences
    and GPT2LMHeadModel/GPT2Model loading of the tGPT checkpoint
- name: PyTorch
  role: Runs the transformer on GPU under torch.no_grad with attention_mask handling
    and mean-pooling of last_hidden_state over token positions 1 through the EOS index
- name: pandas
  role: Assembles the per-patient x per-head entropy table (attention_head_entropy.csv)
    and merges RECIST/molecular event annotations for group testing
provenance:
  source_task_ids:
  - task_008
  - task_007
  source_papers:
  - doi: 10.1016/j.isci.2023.106536
    title: Generative pretraining from large-scale transcriptomes for single-cell
      deciphering
  source_content_hash: sha256:40b029389c27f6fb904c2e50825ace2ab898c1102d9174a0e7f8ab675644252c
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
  - pandas
  - tGPT
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# attention-head-entropy-quantification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantify per-attention-head Shannon entropy and head importance from the 8-layer tGPT transformer applied to bulk tumor transcriptomes, then test whether these head-level features associate with RECIST immunotherapy response, TP53 mutation, HRD, genome doubling, TMB, or survival.

## When to use

Use when bulk tumor transcriptomes must be converted to gene-expression rankings and passed through pretrained tGPT (checkpoint lixiangchun/transcriptome-gpt-1024-8-16-64) with output_attentions=True to extract per-head entropy/importance features for clinical or molecular association testing.

## When NOT to use

- Input is already a per-sample feature/embedding table — this skill requires raw ranked gene-expression sequences to pass through the transformer and collect attention matrices
- Single-cell analysis tasks (clustering, batch integration, trajectory inference) where the standard tGPT feature-extraction plus Leiden/UMAP workflow is the appropriate route rather than attention-head statistics
- Data cannot be tokenized into the 64-token ranked gene format expected by the 'lixiangchun/transcriptome-gpt-1024-8-16-64' checkpoint (e.g. non-transcriptomic measurements)

## Inputs

- Bulk tumor transcriptomes as descending gene-expression ranking sequences (top 64 expressing genes per sample, gene symbols with '-'/'.' replaced by '_')
- RECIST response annotations for the 298-patient urothelial carcinoma ICB cohort (25 CR, 43 PR, 63 SD, 63 PD)
- Molecular event annotations (TP53 mutation, HRD, genome doubling, TMB) and overall survival for the TCGA cohort
- Pretrained tGPT checkpoint 'lixiangchun/transcriptome-gpt-1024-8-16-64' (8 layers, sequence of 62 top-expressing genes, max_len 64)

## Outputs

- attention_head_entropy.csv — per-patient x per-head Shannon entropy/importance table covering all cohort samples
- attention_head_associations.csv — test statistics, p-values, and significant head-cancer/event pairs mirroring Figure 5B-5F
- Group-level summary panels reproducing Figure 5G-5N (per-head entropy distributions stratified by RECIST response group)

## How to apply

Rank genes in each bulk transcriptome by descending expression, replace '-' and '.' with '_' in gene symbols, and tokenize the top 64 genes (max_length=64, truncation and padding enabled) with the PreTrainedTokenizerFast from the tGPT checkpoint 'lixiangchun/transcriptome-gpt-1024-8-16-64'. Run the pretrained GPT2Model/GPT2LMHeadModel on GPU in evaluation mode with output_attentions=True (and output_hidden_states for embeddings) under torch.no_grad, batching 64 samples at a time, collecting attention matrices from all heads of the 8 transformer layers. Compute per-head Shannon entropy and head importance per patient, assembling a per-patient x per-head table (e.g. attention_head_entropy.csv) while mean-pooling last_hidden_state over token positions 1 through the EOS index determined by the attention_mask. Compare head features across groups (e.g. responders CR/PR vs non-responders SD/PD with PD as baseline, or mutated vs wild-type TP53, HRD vs non-HRD) using regression-based testing within each cohort or cancer type. The rationale is that tGPT autoregressively models gene-expression rankings, so attention distributions capture higher-order transcriptome patterns linked to genomic alteration events, prognosis, and immunotherapy outcome.

## Related tools

- **tGPT** (Pretrained transformer (GPT2LMHeadModel/GPT2Model checkpoint lixiangchun/transcriptome-gpt-1024-8-16-64) run with output_attentions=True to supply the 8-layer, per-head attention distributions from ranked bulk transcriptomes) — https://github.com/deeplearningplus/tGPT
- **Transformers** (Provides PreTrainedTokenizerFast for tokenizing the 64-gene ranking sequences and GPT2LMHeadModel/GPT2Model loading of the tGPT checkpoint)
- **PyTorch** (Runs the transformer on GPU under torch.no_grad with attention_mask handling and mean-pooling of last_hidden_state over token positions 1 through the EOS index)
- **pandas** (Assembles the per-patient x per-head entropy table (attention_head_entropy.csv) and merges RECIST/molecular event annotations for group testing)

## Examples

```
tokenizer = PreTrainedTokenizerFast.from_pretrained("lixiangchun/transcriptome-gpt-1024-8-16-64"); model = GPT2LMHeadModel.from_pretrained(checkpoint, output_attentions=True).transformer.to(device).eval()
```

## Evaluation signals

- The entropy table covers all 298 patients of the urothelial carcinoma ICB cohort (25 CR, 43 PR, 63 SD, 63 PD)
- Exactly 5 attention heads are flagged as significantly different between CR/PR and SD/PD groups
- Group-level panels reproduce the gradually varying SD→PR→CR entropy trend reported in Figure 5G-5N, with PD as baseline
- Association results reproduce the reported TCGA pattern: significant head associations with TP53 across 9 cancer types, HRD across 4, genome doubling across 14, and prognostic significance across 7 cancer types
- Attention matrices are collected from all 8 transformer layers, with mean-pooled embeddings taken over token positions 1 through the EOS index defined by the attention_mask

## Limitations

- tGPT uses only the top-expressing genes and may miss information specifically represented in low-expressing genes
- tGPT uses gene expression rankings but not actual expression levels, so fold changes among genes are neglected, which can affect biological interpretation
- The language modeling objective predicts gene rankings, which is not directly related to biological issues; attention-head features therefore require careful validation before clinical interpretation

## Evidence

- [readme] tGPT is conceptually simple in that it autoregressively models the ranking of a gene in the context of its preceding neighbors.: "tGPT is conceptually simple in that it autoregressively models the ranking of a gene in the context of its preceding neighbors."
- [abstract] The feature patterns of tumor bulk tissues learned by tGPT are associated with a wide range of genomic alteration events, prognosis, and treatment outcome of immunotherapy.: "The feature patterns of tumor bulk tissues learned by tGPT are associated with a wide range of genomic alteration events, prognosis, and treatment outcome of immunotherapy."
- [discussion] Attention analysis in relation to signaling via attention-based pathway signaling score: [evidence span withheld — anchor status: not_found]
- [discussion] The input to tGPT was formulated as [<s>, G1, G2, G3, …,<e>]: "The input to tGPT was formulated as [<s>, G1, G2, G3, …,<e>]"
- [readme] model = GPT2LMHeadModel.from_pretrained(checkpoint,output_hidden_states = True).transformer: "model = GPT2LMHeadModel.from_pretrained(checkpoint,output_hidden_states = True).transformer"
