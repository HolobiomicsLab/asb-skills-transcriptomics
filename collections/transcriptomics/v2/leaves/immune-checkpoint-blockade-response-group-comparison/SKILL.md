---
name: immune-checkpoint-blockade-response-group-comparison
description: Use when a bulk-tumor transcriptome cohort has RECIST/ICB response annotations
  and you want to test whether pretrained tGPT attention-head entropy differs between
  responders (CR/PR) and non-responders (SD/PD).
when_to_use_negative:
- Input is single-cell data without bulk-tumor ICB response annotations — tGPT attention-response
  association was established on bulk tumor tissues
- 'You need actual expression-level modeling: tGPT uses expression rankings only,
  so fold changes among genes are neglected and are not recoverable from this skill'
- The cohort lacks outcome/RECIST labels, leaving no groups to compare entropy against
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_2640
tools:
- name: tGPT
  role: Pretrained transformer used to compute attention weights from bulk transcriptome
    expression rankings; checkpoint 'lixiangchun/transcriptome-gpt-1024-8-16-64' with
    8 transformer layers
  repo: https://github.com/deeplearningplus/tGPT
- name: Transformers
  role: Provides PreTrainedTokenizerFast for tokenizing the ranked-gene sequence and
    GPT2Model/GPT2LMHeadModel for loading the checkpoint with output_attentions=True
- name: PyTorch
  role: Runs the model on GPU (device = 'cuda' if available) and supplies DataLoader/Dataset
    for batching ranked-gene lines
- name: pandas
  role: Assembles and saves the per-patient x per-head entropy table (attention_head_entropy.csv)
    and supports group comparison by RECIST response
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.isci.2023.106536
    title: Generative pretraining from large-scale transcriptomes for single-cell
      deciphering
  source_content_hash: sha256:44408c7e4a94ac0eec417173977ee5f197274b796a7ba330d8a233bd11b7ecf2
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
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# immune-checkpoint-blockade-response-group-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Compares tGPT-derived attention-head entropy/importance patterns computed from bulk tumor transcriptomes across RECIST response groups (CR, PR, SD, PD) in an immune checkpoint blockade (ICB) cohort, to identify heads associated with therapy response.

## When to use

Use when a bulk-tumor transcriptome cohort has RECIST/ICB response annotations and you want to test whether pretrained tGPT attention-head entropy differs between responders (CR/PR) and non-responders (SD/PD).

## When NOT to use

- Input is single-cell data without bulk-tumor ICB response annotations — tGPT attention-response association was established on bulk tumor tissues
- You need actual expression-level modeling: tGPT uses expression rankings only, so fold changes among genes are neglected and are not recoverable from this skill
- The cohort lacks outcome/RECIST labels, leaving no groups to compare entropy against

## Inputs

- Bulk tumor transcriptomes (gene expression matrices) from an ICB trial cohort, e.g. 298 urothelial carcinoma patients
- RECIST response annotations per patient (25 CR, 43 PR, 63 SD, 63 PD in the reference cohort)
- Pretrained tGPT checkpoint 'lixiangchun/transcriptome-gpt-1024-8-16-64' (Hugging Face, PreTrainedTokenizerFast + GPT2Model)
- Gene symbols ranked by expression (descending top-expressing genes per sample)

## Outputs

- attention_head_entropy.csv — per-patient x per-head Shannon entropy table covering all cohort patients
- List of attention heads significantly different between CR/PR versus SD/PD (5 in the reference analysis)
- Group-level summary panels (per-head entropy distributions stratified by RECIST response group, Figure 5G-5N style)

## How to apply

Convert each bulk transcriptome into a descending expression-ranking sequence of top-expressing genes, formatted as [<s>, G1, G2, G3, …, <e>], and tokenize it with the PreTrainedTokenizerFast from checkpoint 'lixiangchun/transcriptome-gpt-1024-8-16-64' (max_length 64, truncation and padding enabled). Run the pretrained tGPT GPT2Model on GPU with output_attentions=True to collect attention matrices from all heads of the 8 transformer layers per patient. Compute per-head Shannon entropy and head importance of the attention distributions and assemble a per-patient x per-head entropy table (attention_head_entropy.csv). Compare head entropy between responder (CR/PR) and non-responder (SD/PD) groups using PD as baseline, checking whether entropy varies gradually SD→PR→CR, and flag the significantly different heads (5 in the 298-patient urothelial carcinoma ICB cohort with 25 CR, 43 PR, 63 SD, 63 PD). Render group-level summary panels stratified by RECIST response to reproduce the Figure 5G-5N pattern.

## Related tools

- **tGPT** (Pretrained transformer used to compute attention weights from bulk transcriptome expression rankings; checkpoint 'lixiangchun/transcriptome-gpt-1024-8-16-64' with 8 transformer layers) — https://github.com/deeplearningplus/tGPT
- **Transformers** (Provides PreTrainedTokenizerFast for tokenizing the ranked-gene sequence and GPT2Model/GPT2LMHeadModel for loading the checkpoint with output_attentions=True)
- **PyTorch** (Runs the model on GPU (device = 'cuda' if available) and supplies DataLoader/Dataset for batching ranked-gene lines)
- **pandas** (Assembles and saves the per-patient x per-head entropy table (attention_head_entropy.csv) and supports group comparison by RECIST response)

## Examples

```
from transformers import PreTrainedTokenizerFast, GPT2Model; tok = PreTrainedTokenizerFast.from_pretrained('lixiangchun/transcriptome-gpt-1024-8-16-64'); model = GPT2Model.from_pretrained('lixiangchun/transcriptome-gpt-1024-8-16-64', output_attentions=True)
```

## Evaluation signals

- The entropy table (attention_head_entropy.csv) covers all 298 patients in the reference urothelial carcinoma ICB cohort
- Exactly 5 attention heads are flagged as significantly different between CR/PR and SD/PD
- Group-level panels reproduce the gradually varying SD→PR→CR entropy trend with PD as baseline, as reported in Figure 5G-5N
- Each input transcriptome yields attention matrices from all heads of all 8 transformer layers (output_attentions=True) before entropy is computed
- Tokenization uses max_length=64 with truncation and padding, matching the 62 top-expressing genes sequence of the checkpoint

## Limitations

- tGPT uses only the top-expressing genes; it may miss information specifically represented within low-expressing genes
- tGPT uses gene expression rankings but not actual expression levels, so fold changes among genes are neglected and this can affect biological interpretation
- The language modeling objective predicts gene rankings, which is not directly related to biological issues; further study is required
- Pretrained tGPT weights/checkpoint availability for reuse is only indicated via the code repository link

## Evidence
