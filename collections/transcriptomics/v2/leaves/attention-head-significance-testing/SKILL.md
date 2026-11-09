---
name: attention-head-significance-testing
description: Use when bulk transcriptomes are converted to tGPT gene-ranking token
  sequences and you need to identify which attention heads' entropy patterns differ
  between responder (CR/PR) and non-responder (SD/PD) immunotherapy patients.
when_to_use_negative:
- Input is already a compressed feature/embedding table (e.g. last_hidden_state means)
  rather than raw expression rankings — attention entropy must be computed from attention
  weights, not from extracted features
- Single-cell analysis tasks (clustering, batch integration, trajectory inference)
  — tGPT's single-cell pipeline uses KNN/Leiden on embeddings and does not require
  per-head attention entropy testing
- Data where actual expression magnitudes matter — tGPT's rank-based input neglects
  fold changes among genes, so conclusions about expression-level differences cannot
  come from this skill
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: tGPT
  role: Pretrained transformer whose GPT2Model attention weights (8 layers) are the
    source of per-head entropy computed per patient
  repo: github.com/deeplearningplus/tGPT
- name: Transformers
  role: Provides PreTrainedTokenizerFast for tokenizing 64-token gene-ranking sequences
    and GPT2Model with output_attentions=True
  repo: github.com/deeplearningplus/tGPT
- name: PyTorch
  role: Runs the tGPT model on GPU (cuda device) with torch.no_grad() to collect attention
    matrices for all patients
  repo: github.com/deeplearningplus/tGPT
- name: pandas
  role: Assembles the per-patient x per-head entropy table and supports group-wise
    statistical comparisons between CR/PR and SD/PD
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.isci.2023.106536
    title: Generative pretraining from large-scale transcriptomes for single-cell
      deciphering
  source_content_hash: sha256:50bbb66d1b7b5cc12c53a9c69ad11286425becb89a5bb53840425704e5de107b
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
  - pandas
  - tGPT
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# attention-head-significance-testing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Compute per-attention-head Shannon entropy/importance from the pretrained tGPT GPT2Model applied to bulk tumor transcriptomes, then statistically test which of the 8-layer attention heads discriminate between RECIST response groups in an immune-checkpoint-blockade cohort.

## When to use

Use when bulk transcriptomes are converted to tGPT gene-ranking token sequences and you need to identify which attention heads' entropy patterns differ between responder (CR/PR) and non-responder (SD/PD) immunotherapy patients.

## When NOT to use

- Input is already a compressed feature/embedding table (e.g. last_hidden_state means) rather than raw expression rankings — attention entropy must be computed from attention weights, not from extracted features
- Single-cell analysis tasks (clustering, batch integration, trajectory inference) — tGPT's single-cell pipeline uses KNN/Leiden on embeddings and does not require per-head attention entropy testing
- Data where actual expression magnitudes matter — tGPT's rank-based input neglects fold changes among genes, so conclusions about expression-level differences cannot come from this skill

## Inputs

- Bulk urothelial carcinoma transcriptomes from a 298-patient ICB cohort (25 CR, 43 PR, 63 SD, 63 PD)
- RECIST response annotations per patient (CR/PR/SD/PD labels)
- Gene expression rankings (descending top-expressing gene sequences per patient)
- Pretrained tGPT checkpoint 'lixiangchun/transcriptome-gpt-1024-8-16-64'

## Outputs

- attention_head_entropy.csv — per-patient x per-head Shannon entropy table covering all 298 patients
- List of the 5 attention heads significantly different between CR/PR and SD/PD responders
- Group-level per-head entropy distribution panels stratified by RECIST response group (Figure 5G-5N analog)

## How to apply

Convert each bulk transcriptome into a descending expression-ranking sequence of top-expressing genes and tokenize it with PreTrainedTokenizerFast from the tGPT checkpoint 'lixiangchun/transcriptome-gpt-1024-8-16-64' (max_length=64, truncation and padding enabled), matching the README's LineDataset/DataLoader extraction pattern. Run the pretrained GPT2Model on GPU with output_attentions=True so that attention weight matrices from all heads of the 8 transformer layers are returned per patient, then compute per-head Shannon entropy of each attention distribution and assemble a per-patient x per-head entropy table (attention_head_entropy.csv). The rationale for ranking-based tokenization is tGPT's insensitivity to batch effects and normalization, since it 'takes as input the expression rankings of top-expressing genes rather than the actual expression levels'. Statistically compare head entropy between responder (CR/PR) and non-responder (SD/PD) groups with PD as baseline, checking whether entropy varies gradually from SD to PR to CR, and flag heads significantly different between CR/PR versus SD/PD. Render group-level summary panels (per-head entropy distributions stratified by RECIST response) analogous to Figure 5G-5N.

## Related tools

- **tGPT** (Pretrained transformer whose GPT2Model attention weights (8 layers) are the source of per-head entropy computed per patient) — github.com/deeplearningplus/tGPT
- **Transformers** (Provides PreTrainedTokenizerFast for tokenizing 64-token gene-ranking sequences and GPT2Model with output_attentions=True) — github.com/deeplearningplus/tGPT
- **PyTorch** (Runs the tGPT model on GPU (cuda device) with torch.no_grad() to collect attention matrices for all patients) — github.com/deeplearningplus/tGPT
- **pandas** (Assembles the per-patient x per-head entropy table and supports group-wise statistical comparisons between CR/PR and SD/PD)

## Examples

```
tokenizer = PreTrainedTokenizerFast.from_pretrained("lixiangchun/transcriptome-gpt-1024-8-16-64"); model = GPT2Model.from_pretrained("lixiangchun/transcriptome-gpt-1024-8-16-64", output_attentions=True).to("cuda")
```

## Evaluation signals

- attention_head_entropy.csv covers all 298 patients with entropy values for heads from all 8 transformer layers
- Exactly 5 attention heads are flagged as significantly different between CR/PR and SD/PD groups
- Group-level entropy panels reproduce the gradually varying SD→PR→CR entropy trend with PD as baseline (Figure 5G-5N analog)
- Tokenized sequences use max_length=64 with truncation and padding, matching the 62-top-gene checkpoint
- Attention matrices are obtained with output_attentions=True from GPT2Model (the .transformer module), not from GPT2LMHeadModel logits

## Limitations

- tGPT uses only top-expressing genes, so information in low-expressing genes is missed in the attention patterns
- tGPT uses gene expression rankings but not actual expression levels, so fold changes among genes are neglected and this can affect biological interpretation
- The language modeling objective predicts gene rankings, which is not directly related to biological issues; further study is required for interpretation of head-level signals

## Evidence

- [other] findings from source card: [evidence span withheld — anchor status: not_found]
