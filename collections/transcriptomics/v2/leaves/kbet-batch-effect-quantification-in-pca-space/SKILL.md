---
name: kbet-batch-effect-quantification-in-pca-space
description: Use when integrating public scRNA-seq count matrices (e.g. GSE38495 MII-oocyte,
  GSE45719 E1–E4.0) with low-input bulk RNA-seq and you need to quantify pseudo-bulk
  vs bulk agreement and batch mixing before downstream communication analysis.
when_to_use_negative:
- Your data have no batch labels or only a single batch — kBET requires multiple batches
  to test neighborhood mixing.
- The goal is inferring ligand–receptor cell-cell communication (use TimeTalk, CellChat,
  or CellPhoneDB), not validating the quality of curated expression profiles.
- No stage-matched bulk RNA-seq profiles exist — the correlation component cannot
  be computed and only the kBET batch test applies.
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3361
tools:
- name: kBET
  role: Tests whether data points from different batches are well-mixed within random
    neighborhoods of a fixed size in PCA space
- name: Seurat 3.2.3
  role: Builds Seurat objects and applies cell filtering (CreateSeuratObject, min.features
    = 2000, min.cells = 0) before pseudo-bulk computation
- name: preprocessCore (v1.48.0)
  role: Quantile-normalizes the merged pseudo-bulk + bulk expression matrix via normalize.quantiles
    prior to correlation
- name: TimeTalk
  role: Provides the custom R scripts (via the Figshare deposit) that compute pseudo-bulk
    expression per pre-implantation stage and the QC workflow
  repo: https://github.com/ChengLiLab/TimeTalk
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s42003-023-05283-2
    title: TimeTalk uses single-cell RNA-seq datasets to decipher cell-cell communication
      during early embryo development
  source_content_hash: sha256:93392c3aea33acd00ef3ada5229eb7bca2a5aea39d02db7108a9d70e1eeffee4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ChengLiLab/TimeTalk
  tools:
  - R
  - Seurat
  - Seurat 3.2.3
  - TimeTalk
  - kBET
  - preprocessCore
  - preprocessCore (v1.48.0)
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3361
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# kbet-batch-effect-quantification-in-pca-space

## Summary

A QC validation skill from the TimeTalk workflow: it checks that curated early-embryo scRNA-seq pseudo-bulk profiles agree with stage-matched low-input bulk RNA-seq (Pearson correlation, e.g. 0.91 at the late 2-cell stage) and that batches are well-mixed in PCA space using kBET with a fixed neighborhood size.

## When to use

Use when integrating public scRNA-seq count matrices (e.g. GSE38495 MII-oocyte, GSE45719 E1–E4.0) with low-input bulk RNA-seq and you need to quantify pseudo-bulk vs bulk agreement and batch mixing before downstream communication analysis.

## When NOT to use

- Your data have no batch labels or only a single batch — kBET requires multiple batches to test neighborhood mixing.
- The goal is inferring ligand–receptor cell-cell communication (use TimeTalk, CellChat, or CellPhoneDB), not validating the quality of curated expression profiles.
- No stage-matched bulk RNA-seq profiles exist — the correlation component cannot be computed and only the kBET batch test applies.

## Inputs

- scRNA-seq count matrices from GEO (GSE38495 MII-oocyte, GSE45719 E1–E4.0)
- Low-input bulk RNA-seq profiles from the TimeTalk Figshare deposit (figshare.com/projects/TimeTalk_CB_manuscript_code_data/174498)
- Batch/stage labels for each cell and bulk sample
- Custom TimeTalk R scripts for pseudo-bulk computation

## Outputs

- Per-stage Pearson correlation coefficients between pseudo-bulk scRNA-seq and matched low-input bulk profiles (expect ~0.91 at late 2-cell)
- kBET batch-mixing statistics (acceptance/rejection within random neighborhoods of fixed size in PCA space)
- QC plots reproducing Fig. 1a–b and Supplementary Figs. 1–3

## How to apply

Download the GSE38495 (MII-oocyte) and GSE45719 (E1–E4.0) scRNA-seq count matrices from GEO and the low-input bulk RNA-seq files from the TimeTalk Figshare project deposit. Build Seurat objects with CreateSeuratObject using min.features = 2000 and min.cells = 0, then use the custom R scripts shipped with the TimeTalk deposit to compute pseudo-bulk expression for an average single cell in each pre-implantation stage. Quantile-normalize the merged expression matrix with preprocessCore::normalize.quantiles (preprocessCore version 1.48.0) so pseudo-bulk and bulk profiles are on a comparable scale. Compute Pearson correlation coefficients between each stage's pseudo-bulk profile and the corresponding bulk profile — high agreement is expected (0.91 for the late 2-cell stage). Finally run kBET on the PCA-space representation with a fixed neighborhood size to test whether data points from different batches are well-mixed within random neighborhoods, and plot the agreement and batch-effect results to reproduce Fig. 1a–b and Supplementary Figs. 1–3.

## Related tools

- **kBET** (Tests whether data points from different batches are well-mixed within random neighborhoods of a fixed size in PCA space)
- **Seurat 3.2.3** (Builds Seurat objects and applies cell filtering (CreateSeuratObject, min.features = 2000, min.cells = 0) before pseudo-bulk computation)
- **preprocessCore (v1.48.0)** (Quantile-normalizes the merged pseudo-bulk + bulk expression matrix via normalize.quantiles prior to correlation)
- **TimeTalk** (Provides the custom R scripts (via the Figshare deposit) that compute pseudo-bulk expression per pre-implantation stage and the QC workflow) — https://github.com/ChengLiLab/TimeTalk

## Examples

```
expr.qn <- preprocessCore::normalize.quantiles(expr.merged)
```

## Evaluation signals

- Pearson correlation between stage-matched pseudo-bulk and low-input bulk profiles is high; the paper reports 0.91 for the late 2-cell stage as the expected reference value.
- kBET run in PCA space with a fixed neighborhood size shows batches well-mixed within random neighborhoods (low rejection rate).
- Cell filtering used exactly CreateSeuratObject with min.features = 2000 and min.cells = 0 before pseudo-bulk aggregation.
- Merged expression matrix was quantile-normalized with preprocessCore::normalize.quantiles before computing correlations.
- Plots reproduce Fig. 1a–b and Supplementary Figs. 1–3 of the TimeTalk manuscript.

## Limitations

- kBET's neighborhood size is a fixed user choice; the authors advise users to 'conduct additional data analysis and experiment with various parameter combinations to achieve more dependable results', so results should be sanity-checked across parameter settings.
- The correlation check requires stage-matched bulk profiles; without matched bulk data only the kBET batch-mixing component can be assessed.
- The validation covers pre-implantation mouse stages (MII oocyte GSE38495, E1–E4.0 GSE45719); QC conclusions may not transfer to post-implantation or non-mouse datasets.

## Evidence

- [discussion] The kBET analysis assessed whether the data points with different batches in the PCA space were well-mixed within random neighborhoods of a fixed size.: "The kBET analysis assessed whether the data points with different batches in the PCA space were well-mixed within random neighborhoods of a fixed size."
- [discussion] Quantile normalization was performed with the function “normalize.quantiles” from R package preprocessCore (version 1.48.0).: "Quantile normalization was performed with the function “normalize.quantiles” from R package preprocessCore (version 1.48.0)."
- [discussion] Custom R scripts were used for the pre-implantation scRNA-seq data to calculate the pseudo-bulk expression for an average single cell in each stage.: "Custom R scripts were used for the pre-implantation scRNA-seq data to calculate the pseudo-bulk expression for an average single cell in each stage."
- [discussion] “CreateSeuratObject” in Seurat (parameters: min.features = 2000, min.cells = 0).: "“CreateSeuratObject” in Seurat (parameters: min.features = 2000, min.cells = 0)."
- [readme] we advise users to conduct additional data analysis and experiment with various parameter combinations to achieve more dependable results: "we advise users to conduct additional data analysis and experiment with various parameter combinations to achieve more dependable results"
