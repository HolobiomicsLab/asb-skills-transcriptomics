---
name: luad-therapeutic-candidate-verification
description: Use when a ranked RS-GSFM drug retrieval list for LUAD/A549 is available
  and you must confirm that perhexiline appears among the top-ranked reversal-score
  compounds, consistent with the paper's LUAD candidate identification.
when_to_use_negative:
- The ranked drug list is for a different cancer indication (e.g., BRCA with WYE-354
  or CRPC with NTNCB) rather than LUAD/A549 — the candidate compound and verification
  target differ per cancer type
- The input is a raw gene expression matrix rather than a ranked RS-GSFM reversal-score
  table; the GSFM transformation and RS-GSFM computation must be run first (README
  STEP 2 and STEP 3)
- Normal tissue samples are unavailable so no disease-induced FM pattern can be computed
  (the paper notes the method may not apply to rare diseases without normal samples)
edam_operation: http://edamontology.org/operation_2424
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0154
- http://edamontology.org/topic_2640
tools:
- name: GSFM
  role: Genome-scale functional module transformation framework whose RS-GSFM reversal
    score ranks candidate drugs for LUAD; perhexiline verification is performed against
    its output
  repo: https://github.com/LXY1201/GSFM
- name: signatureSearch R package 1.4.6
  role: Used to download LINCS library drug-induced gene expression profiles that
    feed the RS-GSFM computation underlying the ranked drug list
- name: R 4.1.1
  role: Environment for loading and searching the ranked RS-GSFM drug list and writing
    the confirmation CSV
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.apsb.2024.08.017
    title: 'GSFM: A genome-scale functional module transformation to represent drug
      efficacy for in silico drug discovery'
  source_content_hash: sha256:82323638434d97a413b273c82ba1725d56ab3b792ad2ccc1574175ac17d41c8d
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
  - GSFM
  - R
  - R 4.1.1
  - signatureSearch
  - signatureSearch R package 1.4.6
  edam_operation: http://edamontology.org/operation_2424
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0154
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# luad-therapeutic-candidate-verification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Verification skill that confirms perhexiline as the GSFM-identified candidate therapeutic agent for lung adenocarcinoma by locating it and its RSGSFM reversal score (reported as −0.85) in the ranked RS-GSFM drug retrieval list for the LUAD/A549 condition.

## When to use

Use when a ranked RS-GSFM drug retrieval list for LUAD/A549 is available and you must confirm that perhexiline appears among the top-ranked reversal-score compounds, consistent with the paper's LUAD candidate identification.

## When NOT to use

- The ranked drug list is for a different cancer indication (e.g., BRCA with WYE-354 or CRPC with NTNCB) rather than LUAD/A549 — the candidate compound and verification target differ per cancer type
- The input is a raw gene expression matrix rather than a ranked RS-GSFM reversal-score table; the GSFM transformation and RS-GSFM computation must be run first (README STEP 2 and STEP 3)
- Normal tissue samples are unavailable so no disease-induced FM pattern can be computed (the paper notes the method may not apply to rare diseases without normal samples)

## Inputs

- Ranked RS-GSFM drug retrieval results for the LUAD/A549 condition (drug name, RSGSFM score, rank per compound)
- Disease-induced GSFM active matrix for LUAD derived from TCGA tumor/adjacent-normal expression
- Drug-induced GSFM active matrix from LINCS library profiles downloaded via signatureSearch
- ChEMBL database (version 27) median IC50 data defining the effective compound group (IC50 < 10 μmol/L)

## Outputs

- luad_perhexiline_confirmation.csv containing perhexiline's RSGSFM score, rank position, and verification status
- Verification record asserting perhexiline is present among notable top-ranked RSGSFM compounds for LUAD

## How to apply

Load the ranked RS-GSFM drug retrieval results for the LUAD/A549 condition, where RSGSFM scores were computed with the RS method (adapted from RGES) by comparing drug-induced GSFM active matrix patterns against the disease-induced FM pattern. Search the ranked list for perhexiline and extract its RSGSFM score and rank position relative to the effective (IC50 < 10 μmol/L) compound group defined from ChEMBL version 27 median IC50 data. Verify that perhexiline's placement is consistent with the paper's identification of perhexiline (alongside WYE-354 for BRCA and NTNCB for CRPC) as a candidate therapeutic agent. Write a confirmation record with the RSGSFM score, rank, and verification status to luad_perhexiline_confirmation.csv.

## Related tools

- **GSFM** (Genome-scale functional module transformation framework whose RS-GSFM reversal score ranks candidate drugs for LUAD; perhexiline verification is performed against its output) — https://github.com/LXY1201/GSFM
- **signatureSearch R package 1.4.6** (Used to download LINCS library drug-induced gene expression profiles that feed the RS-GSFM computation underlying the ranked drug list)
- **R 4.1.1** (Environment for loading and searching the ranked RS-GSFM drug list and writing the confirmation CSV)

## Examples

```
run ./project/Notebooks BRCA_GSFM.ipynb
```

## Evaluation signals

- Perhexiline's RSGSFM score in the ranked list matches the reported value of −0.85
- Perhexiline appears among the notable top-ranked RSGSFM compounds for LUAD, consistent with the paper's identification
- Perhexiline's rank position is consistent with membership in the effective compound group (IC50 < 10 μmol/L from ChEMBL v27 median IC50 data)
- The output file luad_perhexiline_confirmation.csv contains the required fields: RSGSFM score, rank, and verification status
- Positive correlation between RSGSFM and IC50 holds across the ranked list, supporting the score as an efficacy measure

## Limitations

- Selecting functional modules from different sources can lead to slight variations in results, though well-aligned disease-related modules keep rankings consistent
- Accurately defining functional modules is challenging when applying GSFM to new datasets without prior knowledge
- The method may not be applicable to rare diseases when normal samples are unavailable
- Additional experimental and clinical studies are needed to validate and elucidate mechanisms of the identified candidates in each individual cancer

## Evidence
