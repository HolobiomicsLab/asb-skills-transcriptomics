---
name: compound-ic50-efficacy-annotation
description: Use when a ranked compound list produced by drug-reversal scoring (RSGSFM
  or similar) needs efficacy context via ChEMBL IC50 annotation to interpret or validate
  top candidates.
when_to_use_negative:
- Compound names in the ranked list cannot be mapped to ChEMBL entries, so no IC50
  values can be retrieved
- The goal is de novo potency prediction rather than annotating already-computed reversal-score
  rankings with literature/ChEMBL efficacy data
- No ranked reversal-score list exists (e.g., input is a raw gene expression matrix
  not yet processed through GSFM transformation and RS scoring)
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3370
tools:
- name: R
  role: Environment (R 4.1.1) in which the ranked compound table is loaded, merged
    with ChEMBL v27 IC50 annotations, and categorized by the 10 μmol/L threshold
  repo: https://github.com/LXY1201/GSFM
- name: dplyr
  role: Used to join/join-filter compound rows and attach IC50 and efficacy-category
    columns to the ranked RSGSFM table
  repo: https://github.com/LXY1201/GSFM
- name: readr
  role: Used to read and write the annotated ranking tables (e.g., brca_rs_gsfm_ranking.csv,
    crpc_ntncb_candidate.csv)
  repo: https://github.com/LXY1201/GSFM
- name: signatureSearch
  role: Provides the upstream drug-induced gene expression profiles from the LINCS
    library that yield the ranked compound lists being annotated
  repo: https://github.com/LXY1201/GSFM
provenance:
  source_task_ids:
  - task_006
  - task_008
  source_papers:
  - doi: 10.1016/j.apsb.2024.08.017
    title: 'GSFM: A genome-scale functional module transformation to represent drug
      efficacy for in silico drug discovery'
  source_content_hash: sha256:32cbce3ef6b27c922d23604909fe7bc8518797afffa61e3b7d6a1fae40d9a7d5
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/LXY1201/GSFM
  repo_url: https://github.com/LXY1201/GSFM
  tools:
  - Docker
  - Jupyter Notebook
  - R
  - dplyr
  - readr
  - signatureSearch
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3370
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# Compound IC50 efficacy annotation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Annotates compounds from a ranked reversal-score list (e.g., RSGSFM) with median IC50 values sourced from the ChEMBL database (version 27) for the disease-relevant cell line, and binarizes efficacy into effective (IC50 < 10 μmol/L) versus ineffective (IC50 > 10 μmol/L) groups. This grounding step supports the paper's premise that RSGSFM correlates with drug efficacy.

## When to use

Use when a ranked compound list produced by drug-reversal scoring (RSGSFM or similar) needs efficacy context via ChEMBL IC50 annotation to interpret or validate top candidates.

## When NOT to use

- Compound names in the ranked list cannot be mapped to ChEMBL entries, so no IC50 values can be retrieved
- The goal is de novo potency prediction rather than annotating already-computed reversal-score rankings with literature/ChEMBL efficacy data
- No ranked reversal-score list exists (e.g., input is a raw gene expression matrix not yet processed through GSFM transformation and RS scoring)

## Inputs

- Ranked compound list with RSGSFM scores (e.g., brca_rs_gsfm_ranking.csv from RS-GSFM drug discovery step)
- ChEMBL database (version 27) median IC50 values per compound for the disease-relevant cell line (e.g., MCF7 for BRCA)
- Compound identifiers/names mappable to ChEMBL entries

## Outputs

- IC50-annotated ranked table (e.g., brca_rs_gsfm_ranking.csv) with median IC50 and effective/ineffective category per compound
- Candidate confirmation table (e.g., crpc_ntncb_candidate.csv) recording compound name, rank, RSGSFM score context, and efficacy annotation

## How to apply

After computing RSGSFM (RS method adapted from RGES, Eqs. 12–16) and ranking compounds, query ChEMBL v27 for each compound's median IC50 against the relevant cell line (e.g., MCF7 for BRCA, PC3 for CRPC) and merge these values onto the ranked table. Categorize compounds as effective when median IC50 < 10 μmol/L and ineffective when IC50 > 10 μmol/L, using this grouping as supporting efficacy context for the RSGSFM ranking. The rationale is the paper's observed positive correlation between RSGSFM and IC50, so top-ranked (most negative RSGSFM) candidates such as WYE-354 (RSGSFM −0.92 for BRCA) and NTNCB (−0.89 for CRPC) should fall in the effective group. Write the annotated ranking to an output table (e.g., brca_rs_gsfm_ranking.csv) and use the effective/ineffective labels to check retrieval performance alongside ROC/enrichment-based accuracyscore evaluation.

## Related tools

- **R** (Environment (R 4.1.1) in which the ranked compound table is loaded, merged with ChEMBL v27 IC50 annotations, and categorized by the 10 μmol/L threshold) — https://github.com/LXY1201/GSFM
- **dplyr** (Used to join/join-filter compound rows and attach IC50 and efficacy-category columns to the ranked RSGSFM table) — https://github.com/LXY1201/GSFM
- **readr** (Used to read and write the annotated ranking tables (e.g., brca_rs_gsfm_ranking.csv, crpc_ntncb_candidate.csv)) — https://github.com/LXY1201/GSFM
- **signatureSearch** (Provides the upstream drug-induced gene expression profiles from the LINCS library that yield the ranked compound lists being annotated) — https://github.com/LXY1201/GSFM

## Evaluation signals

- Annotated output table exists (e.g., brca_rs_gsfm_ranking.csv) and contains an entry for the expected top candidate (WYE-354) with a significant RSGSFM rank and an IC50-derived efficacy category
- Every compound row carries a median IC50 value from ChEMBL v27 for the correct cell line (MCF7 for BRCA, PC3 for CRPC) and a binary effective (IC50 < 10 μmol/L) / ineffective (IC50 > 10 μmol/L) label
- Top-ranked candidates fall into the effective group, consistent with the paper's positive correlation between RSGSFM and IC50
- Reported candidates match paper values: WYE-354 with RSGSFM −0.92 for BRCA and NTNCB with RSGSFM −0.89 for CRPC
- Annotated efficacy categories agree with the ROC/enrichment-based accuracyscore retrieval evaluation (P < 0.05 threshold used in the paper)

## Limitations

- IC50 values depend on the ChEMBL version (v27) and the specific cell line; missing or cell-line-inappropriate IC50s weaken the effective/ineffective grouping
- Computational identification requires additional experimental and clinical studies to validate and elucidate mechanisms of the identified candidates in each individual cancer
- Selecting functional modules from different sources can lead to slight variations in results, which may shift compound ranks and apparent efficacy relationships

## Evidence

- [methods] Efficacy data for each compound (IC50) were sourced from the ChEMBL database (version 27): "Efficacy data for each compound (IC50) were sourced from the ChEMBL database (version 27)"
- [intro] a positive correlation was observed between RSGSFM and IC50: "Notably, a positive correlation was observed between RSGSFM and half-maximal inhibitory concentration (IC50), a key measure of drug efficacy."
- [abstract] RSGSFM could serve as representative measure of drug efficacy: "we found a positive correlation between RSGSFM and drug efficacy, suggesting that RSGSFM could serve as representative measure of drug efficacy"
