---
name: gene-alteration-proportion-zscoring
description: Use when building the CRIT feature matrix for candidate genes and you
  have per-sample TCGA mutation (data_mutations_extended.txt) and CNV (data_CNA.txt)
  files that must be dichotomized into per-gene alteration proportions and Z-scored.
when_to_use_negative:
- You already have a merged feature table (MATRIX.txt) with Zp columns populated —
  recompute only when raw TCGA mutation/CNV inputs change
- You need survival-impact or functional-annotation features (Zs or Zg) — those require
  the survival log-rank or GoSemSim pipelines, not alteration proportion
- Your data are non-cancer or lack per-sample mutation/CNV calls, since Zp depends
  on dichotomized TCGA alteration data across the 19 batches of 12 cancer types
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_2640
- http://edamontology.org/topic_0622
tools:
- name: CRIT
  role: Pipeline whose features-collecting stage computes the Zp alteration-proportion
    Z score from dichotomized TCGA data
  repo: https://github.com/BioinformaticsSTU/CRIT
- name: R
  role: Language in which GET_Prop_zscore.R (Z-Score of alteration in all samples)
    is implemented
- name: biomaRt
  role: Bioconductor dependency installed alongside the CRIT scripts for gene identifier
    handling
  repo: https://bioconductor.org/packages/release/bioc/html/biomaRt.html
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.omtn.2022.10.015
    title: 'CRIT: Identifying RNA-binding protein regulator in circRNA life cycle
      via non-negative matrix factorization'
  source_content_hash: sha256:8bb4139071c3b451d285f09ed32dcf8fadb3fd6e5f84b8f792b57fae0e66e9ab
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/BioinformaticsSTU/CRIT
  repo_url: https://github.com/BioinformaticsSTU/CRIT
  tools:
  - CRIT
  - GoSemSim
  - R
  - biomaRt
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_2640
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# gene-alteration-proportion-zscoring

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes the CRIT 'Zp' feature: a Z score built from dichotomized TCGA mutation and copy-number (CNV) data that reflects the proportion of gene alterations per sample in each cancer type or batch, quantifying how frequently a candidate RBP gene is altered across 19 batches of 12 cancer types.

## When to use

Use when building the CRIT feature matrix for candidate genes and you have per-sample TCGA mutation (data_mutations_extended.txt) and CNV (data_CNA.txt) files that must be dichotomized into per-gene alteration proportions and Z-scored.

## When NOT to use

- You already have a merged feature table (MATRIX.txt) with Zp columns populated — recompute only when raw TCGA mutation/CNV inputs change
- You need survival-impact or functional-annotation features (Zs or Zg) — those require the survival log-rank or GoSemSim pipelines, not alteration proportion
- Your data are non-cancer or lack per-sample mutation/CNV calls, since Zp depends on dichotomized TCGA alteration data across the 19 batches of 12 cancer types

## Inputs

- TCGA per-cancer-type mutation data as tab-separated data_mutations_extended.txt (placed in ./DATA/input/TCGA/cancer/)
- TCGA per-cancer-type copy-number data as tab-separated data_CNA.txt
- TCGA mRNA expression data (data_mRNA_median_Zscores.txt or data_RNA_Seq_v2_mRNA_median_Zscores.txt) and clinical data (data_clinical_patient.txt) as part of the CRIT input directory
- List of candidate genes (RBP_list.RData, 1,344 RBPs with Ensembl/HGNC/Entrez/UniProt identifiers)

## Outputs

- XX_mat_altered.txt: all alteration counts in each batch data of every cancer type
- XX_proportion_Z_Score.txt: feature Zp of each batch data of every cancer type
- Zp column(s) merged into MATRIX.txt (candidate regulators' all features matrix) via GET_feauters_MATRIX.R

## How to apply

For each TCGA cancer type/batch, dichotomize the original mutation and CNV data (data_mutations_extended.txt and data_CNA.txt) so each sample is labeled altered/not altered per gene, then compute the proportion of samples carrying an alteration for each gene. Convert these proportions into per-gene Z scores so alteration frequency is comparable across batches with different sample sizes. Implement this with the CRIT script GET_Prop_zscore.R ('Z-Score of alteration in all samples'), writing per-batch output files named XX_proportion_Z_Score.txt. The resulting Zp columns feed, together with Zs, Zg, rho_m and the C value, into the merged MATRIX.txt used for NMF clustering of 1,344 candidate RBPs. Rationale: the paper assumes a circRNA regulator 'should have at least one of the following alterations, such as point mutation, copy number variation and abnormal mRNA expression in the cancer sample', so alteration burden is a core predictive feature.

## Related tools

- **CRIT** (Pipeline whose features-collecting stage computes the Zp alteration-proportion Z score from dichotomized TCGA data) — https://github.com/BioinformaticsSTU/CRIT
- **R** (Language in which GET_Prop_zscore.R (Z-Score of alteration in all samples) is implemented)
- **biomaRt** (Bioconductor dependency installed alongside the CRIT scripts for gene identifier handling) — https://bioconductor.org/packages/release/bioc/html/biomaRt.html

## Examples

```
Rscript GET_Prop_zscore.R  # after setting TCGA input paths (data_mutations_extended.txt, data_CNA.txt) in CRIT_settings.R; writes XX_mat_altered.txt and XX_proportion_Z_Score.txt per cancer type
```

## Evaluation signals

- Output file XX_proportion_Z_Score.txt exists for each TCGA cancer-type batch after GET_Prop_zscore.R runs
- Zp values are Z-scored (approximately zero mean, unit variance across genes per batch), since they are described as 'a Z score built from the dichotomized original data for each gene'
- Intermediates XX_mat_altered.txt contain per-batch alteration counts consistent with dichotomized mutation and CNV calls
- MATRIX.txt contains feature rows for the 1,344 RBP candidates with the Zp columns populated alongside rho_m, Zs, Zg, and C-value columns
- Alteration status derives from both point mutation and copy-number variation categories, matching the assumption that regulators may harbor 'point mutation, copy number variation and abnormal mRNA expression'

## Limitations

- Zp reflects alteration proportion only within one cancer type or batch; the TCGA compendium spans 19 batches of 12 cancer types, so cross-cancer generalization of a single Zp value requires the merged feature matrix
- The feature set does not by itself establish causality; the paper notes that how dysregulated RBP-circRNA interactions affect patient survival remains an open question
- CRIT's successful application in cancers does not guarantee generalization to other diseases, as flagged in the discussion

## Evidence
