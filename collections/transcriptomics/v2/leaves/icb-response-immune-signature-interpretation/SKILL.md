---
name: icb-response-immune-signature-interpretation
description: Use when bulk tumor RNA-seq profiles from ICB-treated patients have been
  projected through a single-cell-trained encoder and latent features must be tested
  for responder/non-responder association and biologically interpreted.
when_to_use_negative:
- Input data is single-cell expression data rather than bulk tumor RNA-seq — Miscell's
  primary use is single-cell clustering and marker identification, not ICB response
  testing
- The encoder has not been trained on single-cell data, since the method's premise
  is transferring knowledge learned from single-cells to bulk tumor expression data
- Patient response labels are unavailable, since the Wilcoxon rank-sum test requires
  responder/non-responder grouping
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: Miscell
  role: Self-supervised encoder (moco-dim 64) used to project bulk tumor RNA-seq profiles
    into 64 latent features for transfer from single-cell knowledge
  repo: https://github.com/lixiangchun/Miscell
- name: Scanpy
  role: Differential analysis framework whose Wilcoxon rank-sum test is applied to
    score latent-feature association with ICB response
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.isci.2021.103200
    title: 'Miscell: An efficient self-supervised learning approach for dissecting
      single-cell transcriptome'
  source_content_hash: sha256:e699150054604c945064006c23e5d33886339fbffc87ef19754dd9614dfec5f0
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/lixiangchun/Miscell
  repo_url: https://github.com/lixiangchun/Miscell
  tools:
  - Miscell
  - Scanpy
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# icb-response-immune-signature-interpretation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Interprets clinically significant Miscell latent features extracted from bulk tumor RNA-seq by testing their association with immune checkpoint blockade (ICB) response and characterizing significant features via an immune signature (IFNG, IL12B, CXCR3, CD19).

## When to use

Use when bulk tumor RNA-seq profiles from ICB-treated patients have been projected through a single-cell-trained encoder and latent features must be tested for responder/non-responder association and biologically interpreted.

## When NOT to use

- Input data is single-cell expression data rather than bulk tumor RNA-seq — Miscell's primary use is single-cell clustering and marker identification, not ICB response testing
- The encoder has not been trained on single-cell data, since the method's premise is transferring knowledge learned from single-cells to bulk tumor expression data
- Patient response labels are unavailable, since the Wilcoxon rank-sum test requires responder/non-responder grouping

## Inputs

- Bulk tumor RNA-seq expression matrix for the 298-patient urothelial carcinoma ICB cohort (Mariathasan et al., 2018)
- Trained Miscell self-supervised encoder checkpoint (e.g. checkpoint_0199.pth.tar)
- Patient ICB therapy response labels (responder / non-responder)

## Outputs

- icb_response_latent_features.tsv: 64-feature association table (test statistic, raw p-value, adjusted p-value, significance flags)
- List of significant latent features (adjusted p < 0.05)
- Immune signature interpretation of the significant low-scoring feature (IFNG, IL12B, CXCR3, CD19 upregulation)

## How to apply

Project the bulk tumor RNA-seq expression matrix through the trained Miscell self-supervised encoder (moco-dim 64) to obtain 64 latent features per patient. Split the 298 urothelial carcinoma ICB patients (Mariathasan et al., 2018) into responder and non-responder groups and apply the Wilcoxon rank-sum test — the differential analysis adopted by Seurat and Scanpy — to each of the 64 features. Correct the 64 p-values for multiple testing and retain features with adjusted p < 0.05; in the published run exactly 3 of 64 features were significant. For a significant feature with low scores in responders (the 31st feature), interpret it by examining differential expression of immune signature genes: low scores are characterized by upregulation of IFNG, IL12B, CXCR3, and CD19, indicating an activated immune environment. Write the full association table (test statistic, raw and adjusted p-values, significance flags) to icb_response_latent_features.tsv.

## Related tools

- **Miscell** (Self-supervised encoder (moco-dim 64) used to project bulk tumor RNA-seq profiles into 64 latent features for transfer from single-cell knowledge) — https://github.com/lixiangchun/Miscell
- **Scanpy** (Differential analysis framework whose Wilcoxon rank-sum test is applied to score latent-feature association with ICB response)

## Examples

```
features = utils.extract_features(utils.load_pretrained_model(arch, in_features, 'checkpoint_0199.pth.tar', return_feature=True), X)
```

## Evaluation signals

- Exactly 3 of the 64 latent features reach adjusted p < 0.05, matching the reported result
- The significant low-score feature (the 31st) is characterized by upregulation of IFNG, IL12B, CXCR3 and CD19
- The output table icb_response_latent_features.tsv contains all 64 features with test statistic, raw and adjusted p-values, and significance flags
- The Wilcoxon rank-sum test (as adopted by Seurat and Scanpy) is used for the feature-response differential analysis
- Adjusted p-values are computed by correcting the 64 raw p-values for multiple testing

## Limitations

- The clinically significant latent feature extracted by Miscell is lack of explainability and future works are required to address this issue
- The biological mechanisms underlying the latent features of Miscell trained on these single-cell datasets remain unknown and necessitate exploration in future study
- Miscell demands a large sample size to obtain good performance, so small cohorts may not support reliable feature extraction

## Evidence

- [results] ICB association finding: [evidence span withheld — anchor status: not_found]
- [results] immune signature interpretation: [evidence span withheld — anchor status: not_found]
