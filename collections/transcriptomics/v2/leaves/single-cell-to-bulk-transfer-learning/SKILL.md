---
name: single-cell-to-bulk-transfer-learning
description: Use when you have a Miscell encoder trained on single-cell RNA-seq and
  a bulk tumor expression cohort (e.g., Mariathasan ICB or TCGA pan-cancer) whose
  clinical outcomes you want to link to latent features.
when_to_use_negative:
- 'Small datasets: Miscell, analogous to deep neural network methods, demands a large
  sample size to obtain good performance'
- 'Data with uncorrected batch effects: Miscell does not internally include any batch-correction
  methods, so batch correction must be a preprocessing step before projection'
- 'If the goal is explainable clinical markers: the clinically significant latent
  features extracted by Miscell lack explainability and the biological mechanisms
  remain unknown'
tools:
- name: Miscell
  role: Self-supervised encoder whose single-cell-trained latent features are extracted
    for bulk tumor samples (contrastive learning based on MoCo)
  repo: https://github.com/lixiangchun/Miscell
- name: Scanpy
  role: Source of the Wilcoxon rank-sum differential analysis used to test latent
    feature–response associations
- name: MoCo
  role: Source code base for Miscell's contrastive self-supervised learning (moco-t,
    moco-k, moco-m, moco-dim 64)
  repo: https://github.com/facebookresearch/moco
- name: FIt-SNE
  role: t-SNE visualization of latent features in downstream single-cell workflows
    (fast_tsne with perplexity_list=[30, 36, 42, 48])
  repo: https://github.com/KlugerLab/FIt-SNE
provenance:
  source_task_ids:
  - task_007
  - task_008
  source_papers:
  - doi: 10.1016/j.isci.2021.103200
    title: 'Miscell: An efficient self-supervised learning approach for dissecting
      single-cell transcriptome'
  source_content_hash: sha256:bea4ead46a6532b6fed33c0ee8ead087c37a82484ef3af8fc8d5b607f9ca79eb
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
  - FIt-SNE
  - Miscell
  - MoCo
  - Scanpy
  - numpy
  - pandas
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.46153846153846156
n_claims: 13
---

# single-cell-to-bulk-transfer-learning

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Transfer latent features learned by Miscell's self-supervised encoder (contrastive learning on single-cell transcriptomes) to bulk tumor RNA-seq data, then test the 64 latent features for clinical associations such as ICB therapy response and overall survival.

## When to use

Use when you have a Miscell encoder trained on single-cell RNA-seq and a bulk tumor expression cohort (e.g., Mariathasan ICB or TCGA pan-cancer) whose clinical outcomes you want to link to latent features.

## When NOT to use

- Small datasets: Miscell, analogous to deep neural network methods, demands a large sample size to obtain good performance
- Data with uncorrected batch effects: Miscell does not internally include any batch-correction methods, so batch correction must be a preprocessing step before projection
- If the goal is explainable clinical markers: the clinically significant latent features extracted by Miscell lack explainability and the biological mechanisms remain unknown

## Inputs

- Bulk tumor RNA-seq expression matrix (e.g., 298-patient Mariathasan et al. 2018 urothelial carcinoma ICB cohort)
- TCGA pan-cancer bulk expression matrix with clinical table (OS time/status, Age, Sex, TNM stage) joined by TCGA sample barcode
- Pre-trained Miscell encoder checkpoint (e.g., checkpoint_0199.pth.tar, densenet11/densenet21/densenet29/densenet63, --moco-dim 64)

## Outputs

- 64-feature clinical association table (test statistic, raw and adjusted p-values, significance flags), e.g., icb_response_latent_features.tsv
- Cox significant-feature list adjusted for Age/Sex/TNM, e.g., cox_significant_features.csv
- Kaplan-Meier overall survival curves, e.g., km_curve_feature54.png

## How to apply

Project the bulk RNA-seq expression matrix through the pre-trained Miscell encoder (utils.load_pretrained_model with return_feature=True) to obtain 64 latent features per sample; normalize input features before projection as the README recommends. For therapy-response cohorts, split patients into responder vs non-responder groups and apply the Wilcoxon rank-sum test (the differential analysis adopted by Seurat and Scanpy) to each of the 64 features, then correct the 64 p-values for multiple testing and retain features at adjusted p < 0.05. For survival cohorts, fit Kaplan-Meier curves split by the 54th latent feature, run a log-rank test (target p = 0.004), and fit Cox proportional hazards models of overall survival adjusted for Age, Sex, and TNM stage. Export the full 64-feature association table (test statistic, raw and adjusted p-values, significance flags) and the covariate-adjusted significant-feature list.

## Related tools

- **Miscell** (Self-supervised encoder whose single-cell-trained latent features are extracted for bulk tumor samples (contrastive learning based on MoCo)) — https://github.com/lixiangchun/Miscell
- **Scanpy** (Source of the Wilcoxon rank-sum differential analysis used to test latent feature–response associations)
- **MoCo** (Source code base for Miscell's contrastive self-supervised learning (moco-t, moco-k, moco-m, moco-dim 64)) — https://github.com/facebookresearch/moco
- **FIt-SNE** (t-SNE visualization of latent features in downstream single-cell workflows (fast_tsne with perplexity_list=[30, 36, 42, 48])) — https://github.com/KlugerLab/FIt-SNE

## Examples

```
python main.py -a densenet11 -j 1 -b 256 --lr 0.24 --multiprocessing-distributed --in_features 1686 --moco-dim 64 data/x_new_scanpy_HVG.npz
```

## Evaluation signals

- ICB cohort: exactly 3 of the 64 latent features reach adjusted p < 0.05 under the Wilcoxon rank-sum test, matching the reported result
- The significant low-score latent feature (the 31st) is characterized by upregulation of IFNG, IL12B, CXCR3, and CD19
- TCGA: the 54th latent feature's log-rank p-value reproduces the reported p = 0.004
- The covariate-adjusted Cox significant-feature list (54th feature plus 16 others after controlling for Age, Sex, TNM stage) matches the paper's reported listing
- Output tables contain all 64 features with test statistic, raw and adjusted p-values, and significance flags

## Limitations

- Miscell demands a large sample size to obtain good performance, limiting use on small datasets
- Miscell does not internally include any batch-correction methods; users must correct batch effects before running it
- The clinically significant latent features lack explainability and the biological mechanisms underlying them remain unknown
- Cluster-specific marker gene identification is limited to highly variable genes, so marker genes outside HVGs may be missed

## Evidence
