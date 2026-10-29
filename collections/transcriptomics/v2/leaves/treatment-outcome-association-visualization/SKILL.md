---
name: treatment-outcome-association-visualization
description: Use when bulk tumor transcriptomes have been converted to tGPT attention
  features and you need to test and visualize whether per-head entropy/importance
  differs across immunotherapy response groups (CR/PR/SD/PD).
when_to_use_negative:
- Input is single-cell data for clustering or trajectory tasks — the paper's single-cell
  workflow uses embedding features with Leiden clustering, not attention-entropy response
  association
- The cohort lacks clinical response annotations (e.g. RECIST categories), so no responder/non-responder
  grouping can be formed
- Only actual expression levels are available without gene rankings — tGPT requires
  descending expression rankings of top-expressing genes as input
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_2640
tools:
- name: tGPT
  role: Pretrained transformer used to obtain attention weight matrices from all heads
    of 8 layers for bulk tumor transcriptomes
  repo: github.com/deeplearningplus/tGPT
- name: Transformers (PreTrainedTokenizerFast, GPT2Model)
  role: Tokenizes ranked-gene sequences (max_length 64, truncation/padding) and loads
    the GPT2 model with output_attentions=True
- name: PyTorch
  role: Runs the model on GPU with torch.no_grad() and batches samples via DataLoader
- name: pandas
  role: Assembles the per-patient x per-head entropy table (attention_head_entropy.csv)
    and groups samples by RECIST response for statistical comparison and plotting
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.isci.2023.106536
    title: Generative pretraining from large-scale transcriptomes for single-cell
      deciphering
  source_content_hash: sha256:8bcc21a53e27923c2e25189372e7ce4aacb12a85aece4ab71821c1021ded82ef
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
  - Transformers (PreTrainedTokenizerFast, GPT2Model)
  - pandas
  - tGPT
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# treatment-outcome-association-visualization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Links tGPT-derived attention head entropy/importance features from bulk tumor transcriptomes to patient response to immune checkpoint blockade, and renders group-level summary panels (Figure 5G-5N style) showing entropy distributions stratified by RECIST response.

## When to use

Use when bulk tumor transcriptomes have been converted to tGPT attention features and you need to test and visualize whether per-head entropy/importance differs across immunotherapy response groups (CR/PR/SD/PD).

## When NOT to use

- Input is single-cell data for clustering or trajectory tasks — the paper's single-cell workflow uses embedding features with Leiden clustering, not attention-entropy response association
- The cohort lacks clinical response annotations (e.g. RECIST categories), so no responder/non-responder grouping can be formed
- Only actual expression levels are available without gene rankings — tGPT requires descending expression rankings of top-expressing genes as input

## Inputs

- Bulk tumor transcriptomes from the 298-patient urothelial carcinoma ICB cohort with RECIST annotations (25 CR, 43 PR, 63 SD, 63 PD)
- Pretrained tGPT checkpoint 'lixiangchun/transcriptome-gpt-1024-8-16-64' (Hugging Face, PreTrainedTokenizerFast + GPT2Model)
- Gene symbols ranked by expression per sample (text file of descending top-expressing gene rankings)

## Outputs

- Per-patient x per-head attention entropy table saved as attention_head_entropy.csv
- Statistical comparison flagging 5 attention heads significantly different between CR/PR and SD/PD responders
- Group-level summary panels reproducing Figure 5G-5N (per-head entropy distributions by RECIST response group)

## How to apply

Convert each bulk transcriptome into a descending expression-ranking sequence of top-expressing genes, tokenize with PreTrainedTokenizerFast from the checkpoint 'lixiangchun/transcriptome-gpt-1024-8-16-64' (max_length 64, truncation and padding), and run the GPT2Model on GPU with output_attentions=True to collect attention matrices from all heads of the 8 transformer layers. Compute per-head Shannon entropy and head importance per patient, assemble a 298-patient x per-head entropy table (attention_head_entropy.csv), then compare responder (CR/PR, 25+43 patients) versus non-responder (SD/PD, 63+63 patients) groups using PD as baseline. Identify the 5 attention heads significantly different between CR/PR and SD/PD, verify the entropy pattern varies gradually from SD to PR to CR, and render per-head entropy distribution panels stratified by RECIST group to reproduce Figure 5G-5N.

## Related tools

- **tGPT** (Pretrained transformer used to obtain attention weight matrices from all heads of 8 layers for bulk tumor transcriptomes) — github.com/deeplearningplus/tGPT
- **Transformers (PreTrainedTokenizerFast, GPT2Model)** (Tokenizes ranked-gene sequences (max_length 64, truncation/padding) and loads the GPT2 model with output_attentions=True)
- **PyTorch** (Runs the model on GPU with torch.no_grad() and batches samples via DataLoader)
- **pandas** (Assembles the per-patient x per-head entropy table (attention_head_entropy.csv) and groups samples by RECIST response for statistical comparison and plotting)

## Examples

```
tokenizer = PreTrainedTokenizerFast.from_pretrained('lixiangchun/transcriptome-gpt-1024-8-16-64'); batch = tokenizer(lines, max_length=64, truncation=True, padding=True, return_tensors='pt'); x = model(**batch, output_attentions=True)
```

## Evaluation signals

- Entropy table covers all 298 patients (25 CR, 43 PR, 63 SD, 63 PD)
- Exactly 5 attention heads are flagged as significantly different between CR/PR and SD/PD groups
- Group-level panels show the gradually varying entropy pattern ordered SD → PR → CR with PD as baseline, matching Figure 5G-5N
- Attention matrices are returned from all heads of the 8 transformer layers (output_attentions=True) for every sample
- Tokenized sequences respect max_length 64 with truncation and padding applied

## Limitations

- tGPT uses only the top-expressing genes, so information specifically represented within low-expressing genes may be missed
- tGPT uses gene expression rankings but not actual expression levels; fold changes among genes are neglected, which can affect biological interpretation
- The language modeling objective predicts gene rankings, which is not directly related to biological issues; associations with treatment outcome are observational and require further study

## Evidence
