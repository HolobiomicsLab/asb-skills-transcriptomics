---
name: wilcoxon-rank-sum-association-testing
description: Use when bulk tumor expression profiles have been projected through a
  single-cell-trained Miscell encoder (moco-dim 64) and you must test each latent
  feature for responder-vs-non-responder differences in an ICB cohort, or generally
  when comparing two patient groups on non-normal feature scores.
when_to_use_negative:
- Datasets are small — Miscell demands a large sample size to obtain good performance,
  so features from small cohorts will be unreliable
- Batch effects are present and uncorrected — Miscell does not internally include
  batch-correction methods, so running it directly will confound the latent features
- A practitioner needs interpretable features — the clinically significant latent
  features extracted by Miscell lack explainability
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: Miscell
  role: Self-supervised deep neural network encoder (densenet, moco-dim 64) whose
    latent features are extracted from bulk tumor expression data and tested for ICB
    response association
  repo: https://github.com/lixiangchun/Miscell
- name: Scanpy
  role: Source of the differential analysis convention (Wilcoxon rank-sum test) used
    to score feature-response association
- name: Seurat
  role: Reference for the Wilcoxon rank-sum differential analysis approach adopted
    in the feature-response testing
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.isci.2021.103200
    title: 'Miscell: An efficient self-supervised learning approach for dissecting
      single-cell transcriptome'
  source_content_hash: sha256:8afa9c13c37455d07e6c7eb905e8ea89ec12f7fd3586a3dbc9e96af3f7ab72c7
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
  - Seurat
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# wilcoxon-rank-sum-association-testing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Score bulk tumor RNA-seq-derived Miscell latent features (e.g., 64 features per patient from the 298-patient urothelial carcinoma ICB cohort of Mariathasan et al., 2018) for association with immune checkpoint blockade therapy response using the Wilcoxon rank-sum test, the differential analysis adopted by Seurat and Scanpy, followed by multiple-testing correction.

## When to use

Use when bulk tumor expression profiles have been projected through a single-cell-trained Miscell encoder (moco-dim 64) and you must test each latent feature for responder-vs-non-responder differences in an ICB cohort, or generally when comparing two patient groups on non-normal feature scores.

## When NOT to use

- Datasets are small — Miscell demands a large sample size to obtain good performance, so features from small cohorts will be unreliable
- Batch effects are present and uncorrected — Miscell does not internally include batch-correction methods, so running it directly will confound the latent features
- A practitioner needs interpretable features — the clinically significant latent features extracted by Miscell lack explainability

## Inputs

- Bulk tumor RNA-seq expression matrix for 298 urothelial carcinoma ICB patients (Mariathasan et al., 2018)
- Pretrained Miscell encoder checkpoint (e.g., checkpoint_0199.pth.tar) with densenet architecture and 64-dimensional latent space (moco-dim 64)
- Patient ICB response labels (responder vs non-responder)

## Outputs

- icb_response_latent_features.tsv: 64-feature association table with test statistic, raw p-value, adjusted p-value, and significance flags
- Set of significant latent features at adjusted p < 0.05 (reported result: 3 of 64)
- Differential expression check of IFNG, IL12B, CXCR3, CD19 in the low-score group of the significant latent feature (the 31st)

## How to apply

Load the bulk RNA-seq expression matrix (Mariathasan et al., 2018 urothelial carcinoma ICB cohort, 298 patients) and project it through the pretrained Miscell self-supervised encoder to obtain 64 latent features per patient, transferring knowledge learned from single-cells to bulk tumor data. Split patients into ICB responder and non-responder groups, then apply the Wilcoxon rank-sum test — the differential analysis adopted by Seurat and Scanpy — to each of the 64 features. Correct the 64 raw p-values for multiple testing and retain features significant at adjusted p < 0.05. For significant low-scoring features (here the 31st), verify biological plausibility by checking differential expression of the immune signature genes IFNG, IL12B, CXCR3 and CD19 in the low-score patient group, indicating an activated immune environment. Write the full 64-feature table (test statistic, raw and adjusted p-values, significance flags) to icb_response_latent_features.tsv.

## Related tools

- **Miscell** (Self-supervised deep neural network encoder (densenet, moco-dim 64) whose latent features are extracted from bulk tumor expression data and tested for ICB response association) — https://github.com/lixiangchun/Miscell
- **Scanpy** (Source of the differential analysis convention (Wilcoxon rank-sum test) used to score feature-response association)
- **Seurat** (Reference for the Wilcoxon rank-sum differential analysis approach adopted in the feature-response testing)

## Examples

```
model = utils.load_pretrained_model(arch, in_features, 'checkpoint_0199.pth.tar', return_feature=True); features = utils.extract_features(model, X)
```

## Evaluation signals

- Exactly 3 of the 64 latent features reach adjusted p < 0.05, matching the reported result
- The significant low-score latent feature is the 31st and its low-score patient group shows upregulation of IFNG, IL12B, CXCR3 and CD19
- The output icb_response_latent_features.tsv contains exactly 64 rows, each with a Wilcoxon test statistic, raw p-value, adjusted p-value, and significance flag
- Adjusted p-values are computed by correcting the 64 raw Wilcoxon p-values for multiple testing, not left uncorrected

## Limitations

- Miscell demands a large sample size to obtain good performance, limiting use on small datasets
- Miscell does not internally include any batch-correction methods; users must correct batch effects before running it
- The clinically significant latent feature extracted by Miscell lacks explainability; future work is required to address this
- The biological mechanisms underlying the latent features trained on single-cell datasets remain unknown

## Evidence


model = utils.load_pretrained_model(arch, in_features, checkpoint, return_feature=True)

features = utils.extract_features(model, X)"
