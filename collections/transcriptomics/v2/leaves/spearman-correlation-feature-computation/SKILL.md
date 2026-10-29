---
name: spearman-correlation-feature-computation
description: Use when building the CRIT feature matrix and you need a gene-level circRNA
  co-expression score from paired mRNA/circRNA FPKM matrices (e.g., MiOncoCirc fpkm_matrix.csv)
  to quantify gene–circRNA regulatory coupling.
when_to_use_negative:
- The downstream goal is gene alteration burden, survival significance, GO similarity,
  or protein–circRNA binding counts — those are computed by the Zp (GET_Prop_zscore.R),
  Zs (GET_Sur_zscore.R), Zg (GET_Gosim_zscore.R), and C value (GET_C_value.R) scripts
  instead
- You already possess MATRIX.txt or an otherwise complete CRIT feature table; only
  run this if the ρm column is missing or the MiOncoCirc expression data have changed
- The expression data are not paired mRNA/circRNA measurements over the same samples
  (e.g., only TCGA mutation/CNV/clinical tables), since Spearman gene–circRNA correlation
  requires matched expression profiles
edam_operation: http://edamontology.org/operation_3465
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: CRIT
  role: Pipeline within which the ρm feature computation (GET_mRNAcirc_top50cor.R)
    is a step of the 'features collecting' component
  repo: https://github.com/BioinformaticsSTU/CRIT
- name: R
  role: Language used to implement the Spearman rank correlation calculation in GET_mRNAcirc_top50cor.R
  repo: https://github.com/BioinformaticsSTU/CRIT
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.omtn.2022.10.015
    title: 'CRIT: Identifying RNA-binding protein regulator in circRNA life cycle
      via non-negative matrix factorization'
  source_content_hash: sha256:d5589f0edd40589eae0ed3858cdd95927752f85054b11a43ea805c66c1682ff0
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
  edam_operation: http://edamontology.org/operation_3465
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# spearman-correlation-feature-computation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computation of the CRIT feature ρm: for each candidate RBP gene, the mean of the absolute Spearman rank correlation coefficients of its top 50 gene–circRNA pairs, calculated per cancer type from MiOncoCirc FPKM expression data. This feature captures the main influence of a gene on circRNA expression and feeds the feature matrix used for NMF clustering.

## When to use

Use when building the CRIT feature matrix and you need a gene-level circRNA co-expression score from paired mRNA/circRNA FPKM matrices (e.g., MiOncoCirc fpkm_matrix.csv) to quantify gene–circRNA regulatory coupling.

## When NOT to use

- The downstream goal is gene alteration burden, survival significance, GO similarity, or protein–circRNA binding counts — those are computed by the Zp (GET_Prop_zscore.R), Zs (GET_Sur_zscore.R), Zg (GET_Gosim_zscore.R), and C value (GET_C_value.R) scripts instead
- You already possess MATRIX.txt or an otherwise complete CRIT feature table; only run this if the ρm column is missing or the MiOncoCirc expression data have changed
- The expression data are not paired mRNA/circRNA measurements over the same samples (e.g., only TCGA mutation/CNV/clinical tables), since Spearman gene–circRNA correlation requires matched expression profiles

## Inputs

- MiOncoCirc gene expression matrix in FPKM (fpkm_matrix.csv)
- MiOncoCirc circRNA annotation (v0.1.release.txt, all circRNAs filtered at 2 reads in 2,000+ cancer samples, hg38)
- MiOncoCirc sample metadata (meta_update.xlsx, 2,000+ sample annotations) for grouping by cancer type

## Outputs

- Per cancer type feature file XX_symbol_top50_meancor.txt containing the ρm value (mean |Spearman rho| of top 50 gene–circRNA pairs) for each gene
- ρm column contributed to MATRIX.txt, the merged feature matrix of all 1,344 candidate RBP features

## How to apply

Start from the MiOncoCirc compendium files (fpkm_matrix.csv gene expression in FPKM, v0.1.release.txt circRNAs, meta_update.xlsx sample annotations) placed in ./DATA/input/MiOncoCirc. Using GET_mRNAcirc_top50cor.R, for each gene compute Spearman's rank correlation coefficients between the gene's expression and each circRNA's expression within each cancer type, take the absolute values, rank them, select the top 50 gene–circRNA pairs, and average their |rho| values to yield ρm for that gene and cancer type. The rationale is that a candidate regulator of the circRNA life cycle should show strong expression correlation with the circRNAs it influences, so the top-50 mean |rho| summarizes the main influence of the gene on circRNA expression. Run the script with paths configured via CRIT_settings.R, then merge the resulting per-gene ρm values with Zp, Zs, Zg, and C value via GET_feauters_MATRIX.R into MATRIX.txt for the 1,344 RBP candidates. Correctness requires the per-cancer-type output files (XX_symbol_top50_meancor.txt) to exist with populated ρm values for all candidate genes and to merge cleanly into MATRIX.txt.

## Related tools

- **CRIT** (Pipeline within which the ρm feature computation (GET_mRNAcirc_top50cor.R) is a step of the 'features collecting' component) — https://github.com/BioinformaticsSTU/CRIT
- **R** (Language used to implement the Spearman rank correlation calculation in GET_mRNAcirc_top50cor.R) — https://github.com/BioinformaticsSTU/CRIT

## Examples

```
source("CRIT_settings.R"); Rscript CODE/GET_mRNAcirc_top50cor.R  # computes ρm (mean |spearman rho| of top 50 gene-circRNA pairs) per cancer type from MiOncoCirc fpkm_matrix.csv
```

## Evaluation signals

- Running CRIT end to end produces a per-cancer-type output file 'XX_symbol_top50_meancor.txt' for each cancer type in the MiOncoCirc metadata
- Each ρm value lies in the interval [0, 1] because it is a mean of absolute Spearman correlation coefficients
- MATRIX.txt exists and contains feature rows for the 1,344 RBP candidates with the ρm column populated alongside Zp, Zs, Zg, and C value columns
- The ρm values merge 1:1 with gene identifiers (Ensembl ID, HGNC symbol, Entrez ID, UniProt accession) from RBP_list.RData without unmatched genes
- Spot-check one gene by recomputing its mean of the top 50 absolute Spearman correlations per cancer type and confirming it matches the reported ρm

## Limitations

- ρm is computed from MiOncoCirc cancer-sample compendium data (2,000+ samples), so it reflects cancer-context co-expression; generalizing CRIT to other diseases such as neurodegenerative disease would require additional mRNA/circRNA expression profiles under specific conditions and further fine-tuning of the mining algorithms
- Only the top 50 gene–circRNA pairs per gene are averaged, so weak-but-broad correlations outside the top 50 do not contribute to the feature
- The feature is computed per cancer type/batch, so downstream analysis must track which cancer type each ρm value belongs to

## Evidence
