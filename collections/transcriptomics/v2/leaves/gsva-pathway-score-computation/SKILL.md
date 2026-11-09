---
name: gsva-pathway-score-computation
description: Use when a bulk transcriptomic cohort (e.g., GSE39582 or TCGA COAD-READ
  expression matrices) must be scored for a curated pathway list (e.g., high-weight
  pathways from scPAFA/MOFA modules) to build or validate a disease-status classifier.
when_to_use_negative:
- The input is already a single-cell dataset — use scPAFA's 'fast_Ucell' or 'fast_score_genes'
  for cell-level PAS computation instead of GSVA on pseudobulk data
- No pathway dictionary or high-weight pathway selection is available — GSVA scores
  on an arbitrary/unsupervised pathway set will not validate the multicellular modules
- Fewer than 15 samples (not cells) are available — the MOFA-based module identification
  that supplies the high-weight pathways requires at least 15 samples
tools:
- name: GSVA
  role: Computes pathway variation scores for the 30 high-weight module pathways on
    bulk GSE39582 and TCGA COAD-READ expression matrices, converting gene-level expression
    into classifier features
- name: scPAFA
  role: Source of the disease-related multicellular pathway modules; its MOFA factor
    weights define which 30 pathways are scored by GSVA
  repo: https://github.com/ZhuoliHuang/scPAFA
- name: MOFA
  role: Latent factor model whose pathway weights across cell types rank and select
    the high-weight pathways used as GSVA features
  repo: https://github.com/bioFAM/mofapy2
- name: scikit-learn
  role: Trains the L1-regularized LinearSVC (C=0.2) on GSVA scores, predicts on held-out
    and TCGA cohorts, and supplies Platt-scaled probabilities for AUROC
provenance:
  source_task_ids:
  - task_005
  - task_006
  source_papers:
  - doi: 10.1038/s42003-024-07238-7
    title: Uncovering disease-related multicellular pathway modules on large-scale
      single-cell transcriptomes with scPAFA
  source_content_hash: sha256:1e2af063d83205db541b34cc4402fc5385f98b15ec1d08a2f290a3d65b01bd72
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ZhuoliHuang/scPAFA
  tools:
  - GSVA
  - MOFA
  - Python
  - UCell
  - scPAFA
  - scikit-learn
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# gsva-pathway-score-computation

## Summary

Computes GSVA (Gene Set Variation Analysis) pathway scores on bulk RNA expression matrices — here for the 30 highest-MOFA-weight pathways from scPAFA's MMR-related multicellular pathway modules — so they can serve as features for disease-status classifiers such as the L1-regularized SVC predicting MMR/MSI status in CRC.

## When to use

Use when a bulk transcriptomic cohort (e.g., GSE39582 or TCGA COAD-READ expression matrices) must be scored for a curated pathway list (e.g., high-weight pathways from scPAFA/MOFA modules) to build or validate a disease-status classifier.

## When NOT to use

- The input is already a single-cell dataset — use scPAFA's 'fast_Ucell' or 'fast_score_genes' for cell-level PAS computation instead of GSVA on pseudobulk data
- No pathway dictionary or high-weight pathway selection is available — GSVA scores on an arbitrary/unsupervised pathway set will not validate the multicellular modules
- Fewer than 15 samples (not cells) are available — the MOFA-based module identification that supplies the high-weight pathways requires at least 15 samples

## Inputs

- Bulk gene expression matrix for GSE39582 (samples x genes)
- Bulk gene expression matrix for TCGA COAD-READ (samples x genes)
- Pathway dictionary / gene set list: 30 highest-MOFA-weight pathways from the MMR-related multicellular pathway modules
- Sample-level disease/MSI status labels for GSE39582

## Outputs

- GSVA score matrices (samples x 30 pathways) for GSE39582 and TCGA COAD-READ
- Trained L1-regularized LinearSVC (C=0.2, dual=False) fitted on training-split GSVA scores
- Per-cohort AUROC values written to mmr_signature_auroc_results.csv (expected ~0.91 per cohort)

## How to apply

Start from the high-weight pathway set extracted from the scPAFA/MOFA multicellular pathway modules — in the CRC MMR case, the 30 pathways carrying the highest MOFA factor weights. Compute GSVA scores for these pathways on each bulk expression matrix (GSE39582 and the independent TCGA COAD-READ matrix), producing a samples-by-pathways score matrix. Split GSE39582 into 70% training / 30% test partitions, train an L1-regularized LinearSVC on training-split GSVA scores with disease/MSI status labels using C=0.2 selected by 4-fold CV, then predict on the held-out test set and on TCGA COAD-READ, converting decision values to probabilities via Platt scaling and computing AUROC per cohort. The rationale is that GSVA transforms the bulk cohort onto the same pathway-activity feature space as the single-cell-derived modules, allowing the multicellular pathway modules to be validated on independent bulk data.

## Related tools

- **GSVA** (Computes pathway variation scores for the 30 high-weight module pathways on bulk GSE39582 and TCGA COAD-READ expression matrices, converting gene-level expression into classifier features)
- **scPAFA** (Source of the disease-related multicellular pathway modules; its MOFA factor weights define which 30 pathways are scored by GSVA) — https://github.com/ZhuoliHuang/scPAFA
- **MOFA** (Latent factor model whose pathway weights across cell types rank and select the high-weight pathways used as GSVA features) — https://github.com/bioFAM/mofapy2
- **scikit-learn** (Trains the L1-regularized LinearSVC (C=0.2) on GSVA scores, predicts on held-out and TCGA cohorts, and supplies Platt-scaled probabilities for AUROC)

## Evaluation signals

- Held-out 30% GSE39582 test AUROC matches the reported ~0.91 (Fig. 3d-g)
- Independent TCGA COAD-READ test AUROC also matches ~0.91, confirming module generalization across cohorts
- SVC hyperparameter C=0.2 is reproduced from the 4-fold cross-validation on the 70% training partition
- Decision values are converted to probabilities via Platt scaling before AUROC computation, and both per-cohort AUROCs are written to mmr_signature_auroc_results.csv
- Feature matrix dimensionality is exactly 30 pathways (the highest-MOFA-weight set), not the full BioPlanet collection

## Limitations

- Pathway selection is a supervised process; using a redundant pathway set may increase the complexity of the results, while choosing too few or biologically misaligned pathways may cause information loss
- MOFA is a linear model — interpretability is achieved by sacrificing information content in each factor, and interpretability may decrease with multiple disease states
- Pathway-cell type module associations are data-driven, suggesting associations rather than determining biological causal effects
- The scPAFA-derived pathway modules that feed this skill require studies with at least 15 samples (not cells)

## Evidence

- [discussion] These modules are consistent with previous reports in biological interpretation and can further be dissected to extract features for training classifiers.: "These modules are consistent with previous reports in biological interpretation and can further be dissected to extract features for training classifiers."
- [abstract] scPAFA achieves over 40-fold reduction in runtime of PAS computation: "scPAFA can achieve over 40-fold reductions in the runtime of PAS computation"
- [discussion] fast_ucell rank-based PAS is insensitive to dataset composition: "the PAS of “fast_ucell” is based on the ranking of gene expression within each cell, making them insensitive to the dataset in which the cells are located."
- [readme] Pseudobulk PAS matrix reformatted as MOFA long-table input: "In step 2, the single-cell PAS matrix is reformatted into a suitable input ([a long-table-like pandas dataframe]"
- [readme] Downstream statistical analysis of MOFA model with sample-level clinical metadata: "together with sample-level clinical metadata, disease-related multicellular pathway modules (latent factor and its corresponding weights of pathways across cell types) can be identified by"
