---
name: bulk-rna-seq-latent-feature-extraction
description: Use when bulk tumor RNA-seq (e.g. 298-patient urothelial carcinoma ICB
  cohort, Mariathasan et al. 2018) needs biologically meaningful latent features derived
  from a single-cell-trained Miscell model for response association testing.
when_to_use_negative:
- Input is raw single-cell data for clustering and marker gene identification — use
  the standard Miscell scRNA-seq workflow (t-SNE + DBSCAN/Leiden) instead of bulk
  transfer.
- The bulk data has batch effects that have not been corrected — Miscell does not
  internally include any batch-correction methods and requires batch correction as
  a preprocessing step.
- The dataset is very small — Miscell, like other deep-neural-network methods, demands
  a large sample size to obtain good performance.
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: Miscell
  role: Self-supervised encoder (MoCo-style contrastive learning, densenet arch, --moco-dim
    64) whose pretrained checkpoint is applied to bulk expression profiles to extract
    latent features
  repo: github.com/lixiangchun/Miscell
- name: Scanpy
  role: Source of the Wilcoxon rank-sum differential analysis convention used to score
    latent feature association with ICB response
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.isci.2021.103200
    title: 'Miscell: An efficient self-supervised learning approach for dissecting
      single-cell transcriptome'
  source_content_hash: sha256:b0d8091267eeba8244f61f80bbf973ba6a7db145577572c3aaf5ed1f8985a377
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/lixiangchun/Miscell
  tools:
  - Miscell
  - Scanpy
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# bulk-rna-seq-latent-feature-extraction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Transfer a Miscell self-supervised encoder trained on single-cell transcriptomes to bulk tumor RNA-seq by projecting bulk expression profiles through the network to extract 64 latent features, then test each feature's association with clinical phenotypes such as ICB therapy response.

## When to use

Use when bulk tumor RNA-seq (e.g. 298-patient urothelial carcinoma ICB cohort, Mariathasan et al. 2018) needs biologically meaningful latent features derived from a single-cell-trained Miscell model for response association testing.

## When NOT to use

- Input is raw single-cell data for clustering and marker gene identification — use the standard Miscell scRNA-seq workflow (t-SNE + DBSCAN/Leiden) instead of bulk transfer.
- The bulk data has batch effects that have not been corrected — Miscell does not internally include any batch-correction methods and requires batch correction as a preprocessing step.
- The dataset is very small — Miscell, like other deep-neural-network methods, demands a large sample size to obtain good performance.

## Inputs

- Bulk tumor RNA-seq expression matrix for a 298-patient urothelial carcinoma ICB cohort (Mariathasan et al., 2018)
- Pretrained Miscell encoder checkpoint (e.g. checkpoint_0199.pth.tar, densenet arch, --moco-dim 64)
- ICB responder / non-responder patient labels
- Gene list for immune signature validation (IFNG, IL12B, CXCR3, CD19)

## Outputs

- 64-feature latent representation matrix (one row per patient, 64 latent features per patient)
- icb_response_latent_features.tsv: full association table with test statistic, raw and adjusted p-values, and significance flags for all 64 features
- Set of significant features (expected: 3 features at adjusted p < 0.05) with biological annotation of the low-scoring feature 31 (upregulated IFNG, IL12B, CXCR3, CD19)

## How to apply

Load the bulk RNA-seq expression matrix together with the trained Miscell encoder checkpoint (e.g. checkpoint_0199.pth.tar from a densenet architecture trained with --moco-dim 64) and project each bulk profile through the encoder to obtain 64 latent features per patient. Split patients into responder and non-responder groups, and for each of the 64 features apply the Wilcoxon rank-sum test, the differential analysis adopted by Seurat and Scanpy. Correct the 64 p-values for multiple testing and retain features significant at adjusted p < 0.05. For any significant low-scoring feature (here the 31st), verify its biological relevance by checking differential expression of the immune signature genes IFNG, IL12B, CXCR3 and CD19 in low-score patients. Write the full association table (test statistic, raw and adjusted p-values, significance flags) for all 64 features to an output file such as icb_response_latent_features.tsv.

## Related tools

- **Miscell** (Self-supervised encoder (MoCo-style contrastive learning, densenet arch, --moco-dim 64) whose pretrained checkpoint is applied to bulk expression profiles to extract latent features) — github.com/lixiangchun/Miscell
- **Scanpy** (Source of the Wilcoxon rank-sum differential analysis convention used to score latent feature association with ICB response)

## Examples

```
model = utils.load_pretrained_model(arch, in_features, "checkpoint_0199.pth.tar", return_feature=True); features = utils.extract_features(model, X)
```

## Evaluation signals

- Exactly 3 of the 64 latent features reach adjusted p < 0.05 in the Wilcoxon responder vs non-responder test, matching the reported result.
- The latent feature dimension equals 64, consistent with the --moco-dim 64 training parameter of the Miscell encoder.
- The significant low-scoring feature (the 31st) is characterized by upregulation of IFNG, IL12B, CXCR3 and CD19, indicating an activated immune environment.
- Output table contains all 64 features with test statistic, raw p-value, adjusted p-value, and significance flag columns.
- Feature extraction via utils.load_pretrained_model(..., return_feature=True) matches in_features (e.g. 1686 HVGs) between training data and bulk input.

## Limitations

- The clinically significant latent features extracted by Miscell lack explainability; future work is required to address this issue.
- The biological mechanisms underlying the latent features of Miscell trained on single-cell datasets remain unknown and necessitate exploration in future study.
- Miscell demands a large sample size to obtain good performance, analogous to other deep-neural-network-based methods.

## Evidence

- [intro] transferring knowledge learned from single-cells to extract biologically meaningful features from bulk tumor expression data: "transferring knowledge learned from single-cells to extract biologically meaningful features from bulk tumor expression data"
- [discussion] Wilcoxon rank-sum test adopted by Seurat and Scanpy: "differential analysis such as t test and Wilcoxon rank-sum test adopted by Seurat and Scanpy"
- [readme] --moco-dim 64: "--mlp --moco-t 0.2 --moco-k 1024 --moco-m 0.999 --moco-dim 64 $data"
- [discussion] The clinically significant latent feature extracted by Miscell is lack of explainability: "The clinically significant latent feature extracted by Miscell is lack of explainability and future works are required to address this issue."
- [discussion] Miscell does not internally include any batch-correction methods: "Miscell does not internally include any batch-correction methods. Therefore, users are required to correct for batch effects if necessary before running Miscell."
