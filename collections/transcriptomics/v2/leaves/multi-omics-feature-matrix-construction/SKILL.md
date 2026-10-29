---
name: multi-omics-feature-matrix-construction
description: Use when building a feature table that integrates genomic alteration,
  survival, expression-correlation, functional-annotation, and protein-RNA interaction
  data across cancer samples for downstream clustering of candidate regulators.
when_to_use_negative:
- You already have a completed feature matrix (e.g. MATRIX.txt) and only need the
  downstream NMF clustering step (NMF.R) or candidate validation (RBP_analysis.R).
- Your data lack at least one of the required omics layers (TCGA alteration/clinical
  data, MiOncoCirc expression, RNAct interactions, GO annotations), since each of
  the five features depends on a distinct dataset.
- The goal is predicting circRNA-regulating RBPs with supervised learning — the article
  notes this is an unimplemented extension, not part of this feature-construction
  skill.
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_0659
- http://edamontology.org/topic_0099
- http://edamontology.org/topic_2640
- http://edamontology.org/topic_3517
tools:
- name: CRIT
  role: pipeline whose feature-collecting component computes rho_m, Zp, Zs, Zg and
    C value and merges them into the feature matrix
  repo: https://github.com/BioinformaticsSTU/CRIT
- name: survival
  role: R package performing the log-rank tests between samples with or without gene
    alteration that underlie the Zs feature
- name: GoSemSim
  role: R package computing GO semantic similarity scores between genes and GO:0000398
    for the Zg feature
- name: biomaRt
  role: Bioconductor dependency installed alongside the CRIT feature scripts
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.omtn.2022.10.015
    title: 'CRIT: Identifying RNA-binding protein regulator in circRNA life cycle
      via non-negative matrix factorization'
  source_content_hash: sha256:d3795c23673f3f337997d603af3cb83847b1ecd3efaf79b5673293a104be09e9
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
  - survival
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_0659
  - http://edamontology.org/topic_0099
  - http://edamontology.org/topic_2640
  - http://edamontology.org/topic_3517
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# multi-omics-feature-matrix-construction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Constructs the CRIT feature matrix for candidate RBP genes by computing five features — rho_m (top-50 gene-circRNA Spearman correlation), Zp (alteration proportion Z-score from TCGA), Zs (survival p-value Z-score), Zg (GO semantic similarity to GO:0000398), and C value (count of bound circRNAs from RNAct) — merged into one MATRIX.txt for NMF clustering.

## When to use

Use when building a feature table that integrates genomic alteration, survival, expression-correlation, functional-annotation, and protein-RNA interaction data across cancer samples for downstream clustering of candidate regulators.

## When NOT to use

- You already have a completed feature matrix (e.g. MATRIX.txt) and only need the downstream NMF clustering step (NMF.R) or candidate validation (RBP_analysis.R).
- Your data lack at least one of the required omics layers (TCGA alteration/clinical data, MiOncoCirc expression, RNAct interactions, GO annotations), since each of the five features depends on a distinct dataset.
- The goal is predicting circRNA-regulating RBPs with supervised learning — the article notes this is an unimplemented extension, not part of this feature-construction skill.

## Inputs

- TCGA per-cancer-type files: data_mRNA_median_Zscores.txt (or data_RNA_Seq_v2_mRNA_median_Zscores.txt), data_CNA.txt, data_mutations_extended.txt, data_clinical_patient.txt
- MiOncoCirc files: fpkm_matrix.csv (gene FPKM expression), v0.1.release.txt (circRNAs filtered at 2 reads, hg38), meta_update.xlsx (2,000+ sample annotations)
- RNAct files: catrapid_human_basic.zip (catRAPID interaction scores), gencode.v37.annotation.gtf, hsa_hg19_circRNA.txt (CircBase)
- RBP_list.RData: list of 1,344 candidate RBPs with Ensembl gene id, HGNC symbol, Entrez gene id, UniProt SwissProt identifiers
- gold_regulators.xlsx (gold regulators from previous research)

## Outputs

- XX_proportion_Z_Score.txt (feature Zp per batch/cancer type)
- XX_sur_p_value_Z_Score.txt (feature Zs per batch/cancer type)
- GO_sim_Z_Score.txt (feature Zg per gene)
- XX_symbol_top50_meancor.txt (feature rho_m per cancer type)
- C_value_matrix.txt (feature C value)
- MATRIX.txt (merged feature matrix for all candidate regulators, used as NMF input)
- XX_mat_altered.txt (alteration counts per batch)

## How to apply

Run the five GET_*.R scripts to compute each feature: Zp from dichotomized TCGA mutation and CNV data (19 batches of 12 cancer types) via GET_Prop_zscore.R; Zs from log-rank tests (R 'survival' package) between samples with/without gene alteration via GET_Sur_zscore.R, then Z-scored per gene; Zg via GoSemSim as Z-scores of GO semantic similarity between each gene's annotations and GO:0000398 (mRNA splicing, via spliceosome); rho_m from MiOncoCirc FPKM data as the mean absolute Spearman correlation of each gene's top 50 gene-circRNA pairs per cancer type via GET_mRNAcirc_top50cor.R; and C value from RNAct catRAPID interactions filtered to score >= 30, aligning CircBase circRNAs and GENCODE v37 RNAs by chromosomal position, counting circRNAs fully aligned to each protein via GET_C_value.R. Then GET_feauters_MATRIX.R merges all features per RBP gene (identified by Ensembl ID, HGNC symbol, Entrez ID, UniProt accession from RBP_list.RData's 1,344 RBPs) into MATRIX.txt. Configure paths in CRIT_settings.R before sourcing to run the full pipeline. The rationale is that a circRNA regulator should correlate with circRNA expression, harbor alterations, affect survival, relate to spliceosome GO terms, and physically bind circRNAs.

## Related tools

- **CRIT** (pipeline whose feature-collecting component computes rho_m, Zp, Zs, Zg and C value and merges them into the feature matrix) — https://github.com/BioinformaticsSTU/CRIT
- **survival** (R package performing the log-rank tests between samples with or without gene alteration that underlie the Zs feature)
- **GoSemSim** (R package computing GO semantic similarity scores between genes and GO:0000398 for the Zg feature)
- **biomaRt** (Bioconductor dependency installed alongside the CRIT feature scripts)

## Examples

```
Rscript CRIT_settings.R && Rscript GET_Prop_zscore.R && Rscript GET_Sur_zscore.R && Rscript GET_Gosim_zscore.R && Rscript GET_mRNAcirc_top50cor.R && Rscript GET_C_value.R && Rscript GET_feauters_MATRIX.R
```

## Evaluation signals

- MATRIX.txt exists in the output directory and contains feature rows for the 1,344 candidate RBPs with rho_m, Zp, Zs, Zg, and C-value columns populated.
- Per-cancer-type intermediate files (XX_proportion_Z_Score.txt, XX_sur_p_value_Z_Score.txt, XX_symbol_top50_meancor.txt, C_value_matrix.txt, GO_sim_Z_Score.txt) are all written by the corresponding GET_*.R scripts.
- Zp, Zs, and Zg values are Z-scores, so per-feature distributions should be centered near 0; C values are non-negative integer counts of bound circRNAs.
- RNAct interactions were filtered at score >= 30 (weak relationships excluded), so C values should reflect only high-confidence catRAPID bindings with circRNA positions completely aligned to the corresponding RNA.
- Gene identifiers (Ensembl ID, HGNC symbol, Entrez ID, UniProt accession) in the merged matrix match those in RBP_list.RData.

## Limitations

- C value relies on catRAPID-predicted interactions (RNAct) rather than experimental bindings, and requires chromosomal-position alignment between hg19 CircBase circRNAs and GENCODE v37 RNAs.
- rho_m is computed per cancer type from MiOncoCirc circRNAs filtered at 2 reads, so lowly expressed circRNAs may not contribute.
- The pipeline assumes candidates are protein-coding RBPs with GO annotations; genes lacking annotation to GO:0000398-related terms may have uninformative Zg values.
- The README provides an optional RBP_list.RData of 1,344 RBPs; users substituting their own gene list must supply the same four identifier types.

## Evidence

- [methods] rho_m, Zp, Zs, Zg, C: [evidence span withheld — anchor status: external_doc]
