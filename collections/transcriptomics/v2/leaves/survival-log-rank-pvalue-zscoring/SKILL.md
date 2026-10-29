---
name: survival-log-rank-pvalue-zscoring
description: Use when building the CRIT feature matrix for candidate circRNA regulators/RBPs
  and you need to quantify, per gene, how significantly its alterations (mutation/CNV)
  impact TCGA patient survival.
when_to_use_negative:
- A precomputed Zs feature already exists (e.g. XX_sur_p_value_Z_Score.txt) and only
  matrix merging (GET_feauters_MATRIX.R) or NMF clustering remains.
- The cancer dataset lacks patient survival/clinical data (data_clinical_patient.txt),
  so log-rank stratification by alteration status is impossible.
- You need other CRIT features — rho_m (gene-circRNA Spearman correlations from MiOncoCirc),
  Zp (alteration proportion), Zg (GO:0000398 semantic similarity), or C value (RNAct
  catRAPID circRNA-binding counts) — which are computed by their own GET_*.R scripts.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0625
tools:
- name: survival (R package)
  role: Implements the log-rank test between TCGA samples with or without gene alteration
    to derive per-gene survival p-values.
  repo: https://cran.r-project.org/web/packages/survival/survival.pdf
- name: CRIT (GET_Sur_zscore.R)
  role: Pipeline script that performs the survival p-value computation and Z-scoring
    to produce the Zs feature.
  repo: https://github.com/BioinformaticsSTU/CRIT
- name: TCGA
  role: Source of the 19 batches of 12 cancer types providing mutation, CNV, and clinical
    survival data used for the log-rank tests.
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.omtn.2022.10.015
    title: 'CRIT: Identifying RNA-binding protein regulator in circRNA life cycle
      via non-negative matrix factorization'
  source_content_hash: sha256:a28166b9438330c4b00b871783ae471c7cb6260b73bc6917137617075cd675aa
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/BioinformaticsSTU/CRIT
  tools:
  - CRIT
  - CRIT (GET_Sur_zscore.R)
  - GoSemSim
  - R
  - TCGA
  - biomaRt
  - survival
  - survival (R package)
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# survival-log-rank-pvalue-zscoring

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes the CRIT Zs feature: for each gene, a log-rank test (R package 'survival') between TCGA samples with and without the gene alteration is run, and the resulting survival p-values are Z-scored per gene so alteration impact on patient survival can feed the NMF feature matrix.

## When to use

Use when building the CRIT feature matrix for candidate circRNA regulators/RBPs and you need to quantify, per gene, how significantly its alterations (mutation/CNV) impact TCGA patient survival.

## When NOT to use

- A precomputed Zs feature already exists (e.g. XX_sur_p_value_Z_Score.txt) and only matrix merging (GET_feauters_MATRIX.R) or NMF clustering remains.
- The cancer dataset lacks patient survival/clinical data (data_clinical_patient.txt), so log-rank stratification by alteration status is impossible.
- You need other CRIT features — rho_m (gene-circRNA Spearman correlations from MiOncoCirc), Zp (alteration proportion), Zg (GO:0000398 semantic similarity), or C value (RNAct catRAPID circRNA-binding counts) — which are computed by their own GET_*.R scripts.

## Inputs

- TCGA tab-separated mutation data per cancer type: data_mutations_extended.txt
- TCGA tab-separated CNV data per cancer type: data_CNA.txt
- TCGA tab-separated clinical survival data per patient: data_clinical_patient.txt
- RBP_list.RData candidate gene list (1,344 RBPs) to restrict scoring

## Outputs

- Per-batch Zs feature file: XX_sur_p_value_Z_Score.txt (feature Zs of each batch data of every cancer type)
- Zs column contributed to the merged feature matrix MATRIX.txt

## How to apply

Dichotomize TCGA samples within each of the 19 batches of 12 cancer types into altered vs. non-altered groups based on mutation (data_mutations_extended.txt) and CNV (data_CNA.txt) status. Run a log-rank test between the two groups using the R package 'survival' with the clinical survival data (data_clinical_patient.txt), yielding a p-value per gene reflecting how strongly the gene's alteration affects patient survival. Z-score these survival p-values across genes within each batch to produce the Zs feature, via the CRIT script GET_Sur_zscore.R. The rationale is that authentic circRNA regulators should harbor alterations with measurable clinical consequences, making Zs one of the weighted features (alongside rho_m, Zp, Zg, C) merged by GET_feauters_MATRIX.R for NMF clustering.

## Related tools

- **survival (R package)** (Implements the log-rank test between TCGA samples with or without gene alteration to derive per-gene survival p-values.) — https://cran.r-project.org/web/packages/survival/survival.pdf
- **CRIT (GET_Sur_zscore.R)** (Pipeline script that performs the survival p-value computation and Z-scoring to produce the Zs feature.) — https://github.com/BioinformaticsSTU/CRIT
- **TCGA** (Source of the 19 batches of 12 cancer types providing mutation, CNV, and clinical survival data used for the log-rank tests.)

## Examples

```
Rscript GET_Sur_zscore.R  # after sourcing CRIT_settings.R with TCGA input paths
```

## Evaluation signals

- The expected output file XX_sur_p_value_Z_Score.txt exists in the output directory for each TCGA batch/cancer type after running GET_Sur_zscore.R.
- Every candidate gene in the 1,344-RBP list has a defined Zs value within each batch (no missing rows for genes with computed log-rank tests).
- Zs values are Z-scored within each batch (approximately zero mean, unit standard deviation across genes), not raw log-rank p-values.
- The merged MATRIX.txt contains a populated Zs column for all 1,344 RBP candidates, confirming the feature was merged by GET_feauters_MATRIX.R.

## Limitations

- Zs is only meaningful where TCGA clinical survival data exist for the batch; alteration-stratified log-rank tests require dichotomizable mutation/CNV status per sample.
- The feature reflects alteration effects only within one cancer type or batch at a time (19 batches of 12 cancer types), so cross-cancer generalization requires recomputation per batch.
- Survival analysis captures only clinical significance of alterations; it does not measure circRNA expression correlation (rho_m), GO splicing similarity (Zg), or physical circRNA binding (C value), which must come from the other feature scripts.

## Evidence
