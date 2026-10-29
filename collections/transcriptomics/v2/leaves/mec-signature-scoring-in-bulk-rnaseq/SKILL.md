---
name: mec-signature-scoring-in-bulk-rnaseq
description: Use when you need to project a single-cell-derived MetaTiME signature
  onto bulk tumor RNA-seq (e.g. TCGA with clinical survival metadata) and test whether
  higher MeC expression associates with prognosis.
when_to_use_negative:
- Input is single-cell TME data to be annotated — use the MetaTiME-Annotator cell-state
  annotation workflow instead of bulk signature scoring
- The goal is de novo discovery of gene programs rather than scoring a pretrained
  MeC signature — MetaTiME re-training pipeline (MetaTiMEpretrain) is the appropriate
  route
- No clinical outcome metadata (e.g. survival time/status) is available, since the
  outcome association test cannot be performed
edam_operation: http://edamontology.org/operation_0314
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3517
tools:
- name: MetaTiME
  role: Provides the pretrained MeC gene weights (metatime.mecs), bulk data loading
    (metatime.loaddata), and mapping of samples onto the MeC space (metatime.mecmapper
    / metatime.plmapper)
  repo: https://github.com/yi-zhang/MetaTiME
- name: lifelines
  role: Python library used to fit Kaplan–Meier survival curves and run log-rank tests
    comparing high vs low MeC-42 signature groups
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-023-38333-8
    title: MetaTiME integrates single-cell gene expression to characterize the meta-components
      of the tumor immune microenvironment
  source_content_hash: sha256:5d88db02e0d0b4b2aa7a4d41d2223f915d45dc091b685966088c53ff03599e9f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/yi-zhang/MetaTiME
  tools:
  - MetaTiME
  - lifelines
  edam_operation: http://edamontology.org/operation_0314
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3517
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# mec-signature-scoring-in-bulk-rnaseq

## Summary

Scores a MetaTiME meta-component (MeC) signature, e.g. MeC-42 (M_Macrophage-IL1-NFKB), in bulk RNA-seq tumors by averaging the top 20 genes ranked by the pretrained MeC z-weight, then tests the score's association with clinical outcomes such as TCGA survival via Kaplan–Meier curves and log-rank tests.

## When to use

Use when you need to project a single-cell-derived MetaTiME signature onto bulk tumor RNA-seq (e.g. TCGA with clinical survival metadata) and test whether higher MeC expression associates with prognosis.

## When NOT to use

- Input is single-cell TME data to be annotated — use the MetaTiME-Annotator cell-state annotation workflow instead of bulk signature scoring
- The goal is de novo discovery of gene programs rather than scoring a pretrained MeC signature — MetaTiME re-training pipeline (MetaTiMEpretrain) is the appropriate route
- No clinical outcome metadata (e.g. survival time/status) is available, since the outcome association test cannot be performed

## Inputs

- Pretrained MetaTiME MeC gene-weight table (from metatime.mecs)
- TCGA bulk RNA-seq tumor expression matrix (via metatime.loaddata)
- TCGA clinical survival metadata (time-to-event and status per sample)

## Outputs

- Per-sample MeC signature scores (e.g. tcga_mec42_scores.csv)
- Kaplan–Meier survival curves comparing high vs low signature groups (e.g. tcga_mec42_km_survival.png)
- Log-rank test statistics, overall and stratified by cancer type

## How to apply

Load the pretrained MeC gene-weight table via the metatime.mecs module and extract the top 20 genes ranked by z-weight for the MeC of interest (e.g. MeC-42, M_Macrophage-IL1-NFKB). Load TCGA bulk RNA-seq expression and clinical survival metadata with metatime.loaddata, and map bulk samples onto the MeC space using metatime.mecmapper / metatime.plmapper. Score every tumor as the averaged expression of the 20 signature genes, split tumors into high- and low-signature groups, and fit Kaplan–Meier curves in Python (lifelines), comparing groups with a log-rank test. Repeat the analysis stratified by cancer type to identify which tumor types drive the association (e.g. LGG and KIRC). Persist per-sample scores (tcga_mec42_scores.csv) and KM curves (tcga_mec42_km_survival.png) for validation.

## Related tools

- **MetaTiME** (Provides the pretrained MeC gene weights (metatime.mecs), bulk data loading (metatime.loaddata), and mapping of samples onto the MeC space (metatime.mecmapper / metatime.plmapper)) — https://github.com/yi-zhang/MetaTiME
- **lifelines** (Python library used to fit Kaplan–Meier survival curves and run log-rank tests comparing high vs low MeC-42 signature groups)

## Examples

```
import metatime.mecs as mecs; genes = mecs.load_mec_weights().nlargest(20, 'MeC-42').index; scores = tcga_expr[genes].mean(axis=1)
```

## Evaluation signals

- Reproduced Kaplan–Meier curves show higher MeC-42 (IL1B/NFKB macrophage) signature expression associates with lower survival, matching the direction of the published finding
- The effect is most pronounced in LGG and KIRC when the analysis is stratified by cancer type
- Exactly 20 genes are extracted, ranked by z-weight from the MeC-42 gene-weight table
- Per-sample scores file (tcga_mec42_scores.csv) contains a score for every TCGA tumor sample, and the KM figure (tcga_mec42_km_survival.png) is saved
- Log-rank test p-values are reported for both the pan-cancer and per-cancer-type analyses

## Limitations

- The MeC space was learnt from tumor microenvironment single cells across hundreds of tumor scRNA-seq datasets, so scoring bulk tumors is an approximation that assumes signature genes capture the same biology in bulk mixtures
- Association with survival is correlative and tumor-type dependent; the signal is strong in LGG and KIRC but may be absent or reversed in other cancer types
- Bulk expression confounds tumor-cell and microenvironment contributions, so the score reflects mixture-level IL1B/NFKB macrophage activity rather than single-cell state assignments

## Evidence

- [readme] MetaTiME learns data-driven, interpretable, and reproducible gene programs by integrating millions of single cells from hundreds of tumor scRNA-seq data.: "MetaTiME learns data-driven, interpretable, and reproducible gene programs by integrating millions of single cells from hundreds of tumor scRNA-seq data."
- [intro] MeCs as 86 transcriptional directions: "These MeCs represent the TME landscape along 86 data-driven transcriptional directions mirroring lineage-specific cell states and signaling activities."
- [abstract] Projection onto MetaTiME space: "By projecting onto the MetaTiME space, we provide a tool to annotate cell states and signature continuums"
- [discussion] Toolkit for analyzing independent TME datasets: "MetaTiME provides a toolkit for analyzing independent TME scRNA-seq datasets by mapping gene expression onto the MeC space."
