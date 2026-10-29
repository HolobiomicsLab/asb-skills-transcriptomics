---
name: cancer-multiomics-feature-integration
description: Use when you need to merge gene alteration, survival, GO-similarity,
  and protein-RNA binding evidence for candidate RBPs into one feature matrix before
  unsupervised clustering of circRNA regulators.
when_to_use_negative:
- You already have a merged candidate regulator feature table (MATRIX.txt) and only
  need downstream clustering or validation steps
- Your data are not cancer samples with matched TCGA alteration/clinical data and
  circRNA expression profiles — the article flags generalization to other diseases
  (e.g., neurodegenerative disease) as untested
- You need supervised prediction of circRNA-regulating RBPs from known regulators
  — the article lists this as an unimplemented potential extension, not what this
  unsupervised pipeline does
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0203
tools:
- name: CRIT
  role: NMF-based pipeline whose GET_*.R scripts compute and merge the five features
    into MATRIX.txt and cluster candidate RBPs
  repo: https://github.com/BioinformaticsSTU/CRIT
- name: R
  role: Implementation language for feature scripts, NMF clustering, and Fisher's
    exact validation tests
- name: NMF
  role: R package used by NMF.R to cluster candidate RBPs at rank R=36 selected via
    CPCC and DC indices
- name: survival
  role: R package implementing the log-rank tests underlying the Zs survival feature
- name: GoSemSim
  role: R package computing GO similarity to GO:0000398 for the Zg feature
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.omtn.2022.10.015
    title: 'CRIT: Identifying RNA-binding protein regulator in circRNA life cycle
      via non-negative matrix factorization'
  source_content_hash: sha256:cedd25331daa11053e473c0d465d4681d05d6d94e87e5c357eb8aca177035be6
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
  - NMF
  - R
  - survival
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 15
---

# cancer-multiomics-feature-integration

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Builds the CRIT feature matrix by integrating five multi-omics features (ρm, Zp, Zs, Zg, C value) for 1344 candidate RBPs across 12 TCGA cancer types and the MiOncoCirc compendium, then clusters them via NMF to surface circRNA-regulator-enriched clusters. It is the core data-integration step that turns heterogeneous genomics, transcriptomics, and protein-RNA interactomics evidence into a single matrix suitable for unsupervised regulator discovery.

## When to use

Use when you need to merge gene alteration, survival, GO-similarity, and protein-RNA binding evidence for candidate RBPs into one feature matrix before unsupervised clustering of circRNA regulators.

## When NOT to use

- You already have a merged candidate regulator feature table (MATRIX.txt) and only need downstream clustering or validation steps
- Your data are not cancer samples with matched TCGA alteration/clinical data and circRNA expression profiles — the article flags generalization to other diseases (e.g., neurodegenerative disease) as untested
- You need supervised prediction of circRNA-regulating RBPs from known regulators — the article lists this as an unimplemented potential extension, not what this unsupervised pipeline does

## Inputs

- MiOncoCirc fpkm_matrix.csv and v0.1.release.txt circRNA expression (hg38)
- TCGA per-cancer data_mRNA_median_Zscores.txt, data_CNA.txt, data_mutations_extended.txt, data_clinical_patient.txt
- RNAct catrapid_human_basic.zip interactome scores
- gencode.v37.annotation.gtf and hsa_hg19_circRNA.txt chromosomal annotations
- gold_regulators.xlsx list of gold-standard circRNA regulators
- RBP_list.RData (1344 candidate RBPs)

## Outputs

- XX_proportion_Z_Score.txt (Zp per batch/cancer type)
- XX_sur_p_value_Z_Score.txt (Zs per batch/cancer type)
- GO_sim_Z_Score.txt (Zg per gene)
- XX_symbol_top50_meancor.txt (ρm per cancer type)
- C_value_matrix.txt (C value per cancer type)
- MATRIX.txt (merged feature matrix)
- NMFgroup_XXclusterYYrun.txt (cluster assignments per rank and run)

## How to apply

Compute five features per candidate gene: ρm (mean absolute Spearman correlation of the top 50 gene-circRNA pairs from MiOncoCirc fpkm_matrix.csv), Zp (Z-score of gene alteration proportion across 19 TCGA batches of 12 cancer types, from CNV/mutation/mRNA files), Zs (Z-score of log-rank survival p-values from TCGA clinical data via the R 'survival' package), Zg (Z-score of GO similarity to GO:0000398 'mRNA splicing, via spliceosome' via R 'GoSemSim'), and C value (number of circRNAs fully aligned to each protein's RNA, from RNAct/catRAPID data with interactions scoring <30 excluded, using gencode.v37.annotation.gtf and hsa_hg19_circRNA.txt positions). Merge all features into MATRIX.txt with GET_feauters_MATRIX.R, restricting to the 1344 candidate RBPs in RBP_list.RData. Then run NMF (NMF.R) on MATRIX.txt, selecting rank R=36 by local maxima of the cophenetic correlation coefficient (CPCC) and dispersion (DC) indices, and write per-run assignments as NMFgroup_XXclusterYYrun.txt. Decision point: identify the gold-regulator-enriched cluster by Fisher's exact test against gold_regulators.xlsx; a correct run reproduces cluster 23 with 90 RBPs including 17 gold-standard regulators (p=0.00012).

## Related tools

- **CRIT** (NMF-based pipeline whose GET_*.R scripts compute and merge the five features into MATRIX.txt and cluster candidate RBPs) — https://github.com/BioinformaticsSTU/CRIT
- **R** (Implementation language for feature scripts, NMF clustering, and Fisher's exact validation tests)
- **NMF** (R package used by NMF.R to cluster candidate RBPs at rank R=36 selected via CPCC and DC indices)
- **survival** (R package implementing the log-rank tests underlying the Zs survival feature)
- **GoSemSim** (R package computing GO similarity to GO:0000398 for the Zg feature)

## Examples

```
Rscript GET_feauters_MATRIX.R && Rscript NMF.R
```

## Evaluation signals

- MATRIX.txt contains all five features (ρm, Zp, Zs, Zg, C value) for the 1344 RBPs in RBP_list.RData
- NMF rank R=36 is supported by local maxima of both the cophenetic correlation coefficient (CPCC) and dispersion (DC) indices
- Enriched cluster contains exactly 90 RBPs including 17 gold-standard circRNA regulators
- Fisher's exact p-values reproduce reported values: 0.00012 (gold regulators), 0.00201 (13/90 overlap with Li et al.'s 103-RBP circmCherry screen), 0.00112 (6/19 overlap with curated established regulators)
- C value feature respects the RNAct interaction score ≥30 cutoff (weak interactions with score <30 excluded)

## Limitations

- Generalization beyond cancers (e.g., neurodegenerative disease) is untested per the article discussion
- Cross-disease analysis would require fine-tuned mining algorithms and additional mRNA/circRNA expression profiles under specific conditions
- Supervised-learning-based prediction of circRNA-regulating RBPs remains a potential unimplemented extension
- Pipeline depends on chromosomal-position alignment of circRNAs (CircBase hg19, GENCODE v37) to RNAct RNAs, so annotation version mismatches can break the C value feature

## Evidence

- [intro] CRIT integrates gene alteration, clinical significance, gene-circRNA expression correlation, and RBP-RNA-binding data into a statistical framework.: [evidence span withheld — anchor status: not_found]
- [methods] C value feature definition: "C value was defined as the number of circRNA with which each protein binds"
- [methods] RNAct weak-interaction cutoff: "From the RNAct database, excluding the weak relationship with interaction score<30 of the protein-RNA interaction data"
- [methods] NMF clustering rationale: "CRIT has been designed to identify circRNA regulators by using an NMF-based pipeline"
- [readme] Feature merge script: ""GET_feauters_MATRIX.R" (merge feauters to a matrix)"
- [readme] Cluster output naming convention: ""NMFgroup_XXclusterYYrun.txt": cluster result of number XX of cluster and YY times run."
