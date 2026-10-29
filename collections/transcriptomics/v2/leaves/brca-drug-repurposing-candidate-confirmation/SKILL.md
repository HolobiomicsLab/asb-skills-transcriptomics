---
name: brca-drug-repurposing-candidate-confirmation
description: Use when a BRCA (MCF7) GSFM active matrix and drug-induced GSFM matrix
  are available and the goal is to rank compounds by RSGSFM and confirm a reported
  top candidate such as WYE-354 against IC50 and FDA-approved/investigational drug
  benchmarks.
when_to_use_negative:
- Input data are raw gene expression matrices rather than GSFM-transformed functional
  module active matrices (run the GSFM transformation first)
- The disease of interest is not BRCA or lacks matched normal samples — the article
  notes the method may not be applicable to certain rare diseases when normal samples
  are unavailable
- Functional modules cannot be reasonably defined for the new dataset without prior
  knowledge, which the discussion flags as a challenge for applying GSFM
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: GSFM
  role: Framework producing the FM active matrices and providing the RSGSFM reversal-score
    computation for candidate ranking
  repo: https://github.com/LXY1201/GSFM
- name: signatureSearch
  role: R package (1.4.6) used to download LINCS drug-induced gene expression profiles
    feeding the GSFM matrices
- name: R
  role: R 4.1.1 environment (with pROC 1.17.0.1, ROCR 1.0.11) for RS calculation,
    ROC-based 10-fold cross-validation and enrichment analysis
- name: Jupyter Notebook
  role: Runs BRCA_GSFM.ipynb for the BRCA GSFM transformation step inside the GSFM-notebook
    container
- name: Docker
  role: gsfm-script:latest image running the GSFM-notebook container that hosts the
    notebooks
  repo: https://github.com/LXY1201/GSFM
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.apsb.2024.08.017
    title: 'GSFM: A genome-scale functional module transformation to represent drug
      efficacy for in silico drug discovery'
  source_content_hash: sha256:5957422e1411e03ee555aeda2a0f86630e86f7c9eace08ed697c3552bc3460d5
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
  - GSFM
  - Jupyter Notebook
  - R
  - signatureSearch
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# brca-drug-repurposing-candidate-confirmation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Confirms WYE-354 (RSGSFM −0.92) as the top-ranked candidate therapeutic agent for breast-invasive carcinoma (BRCA) by computing the RS reversal score on GSFM-transformed drug- and disease-induced functional-module (FM) active matrices and validating retrieval performance with ROC-based cross-validation and IC50-based enrichment.

## When to use

Use when a BRCA (MCF7) GSFM active matrix and drug-induced GSFM matrix are available and the goal is to rank compounds by RSGSFM and confirm a reported top candidate such as WYE-354 against IC50 and FDA-approved/investigational drug benchmarks.

## When NOT to use

- Input data are raw gene expression matrices rather than GSFM-transformed functional module active matrices (run the GSFM transformation first)
- The disease of interest is not BRCA or lacks matched normal samples — the article notes the method may not be applicable to certain rare diseases when normal samples are unavailable
- Functional modules cannot be reasonably defined for the new dataset without prior knowledge, which the discussion flags as a challenge for applying GSFM

## Inputs

- BRCA (MCF7) drug-induced GSFM active matrix (from STEP 2 GSFM transformation)
- BRCA disease GSFM active matrix from TCGA tumor/adjacent-normal expression (GSE3744, GSE42568, GSE109169 used for BRCA)
- ChEMBL database (version 27) median IC50 values for MCF7
- FDA-approved or investigational BRCA drug list for enrichment analysis

## Outputs

- brca_rs_gsfm_ranking.csv: compounds ranked by RSGSFM with IC50 annotations and effective/ineffective categories
- ROC-based 10-fold cross-validation results and enrichment P-values/accuracyscore (r × (−log(P_Enrichment)))
- Confirmation of WYE-354 as a significant top-ranked BRCA candidate (RSGSFM ≈ −0.92)

## How to apply

Load the BRCA (MCF7) drug-induced GSFM active matrix and the BRCA disease GSFM active matrix produced by the GSFM transformation (BRCA_GSFM.ipynb), then compute the RS method (adapted from RGES) for each compound by deriving esup and esdown from ranked FM lists and scoring RSGSFM = esup − esdown (Eqs. 12–16). Rank all compounds by RSGSFM and annotate them with median IC50 values from ChEMBL v27 for MCF7, categorizing compounds as effective (IC50 < 10 μmol/L) or ineffective (IC50 > 10 μmol/L); the article reports a positive correlation between RSGSFM and IC50, so more negative scores indicate better expected efficacy. Evaluate retrieval performance with ROC-based 10-fold cross-validation (pROC/ROCR) and enrichment analysis using FDA-approved or investigational BRCA drugs, computing the accuracyscore r × (−log(P_Enrichment)). Confirm that WYE-354 appears among the significant top-ranked candidates with a strongly negative RSGSFM (−0.92 in the paper), and write the ranked table to brca_rs_gsfm_ranking.csv.

## Related tools

- **GSFM** (Framework producing the FM active matrices and providing the RSGSFM reversal-score computation for candidate ranking) — https://github.com/LXY1201/GSFM
- **signatureSearch** (R package (1.4.6) used to download LINCS drug-induced gene expression profiles feeding the GSFM matrices)
- **R** (R 4.1.1 environment (with pROC 1.17.0.1, ROCR 1.0.11) for RS calculation, ROC-based 10-fold cross-validation and enrichment analysis)
- **Jupyter Notebook** (Runs BRCA_GSFM.ipynb for the BRCA GSFM transformation step inside the GSFM-notebook container)
- **Docker** (gsfm-script:latest image running the GSFM-notebook container that hosts the notebooks) — https://github.com/LXY1201/GSFM

## Examples

```
docker run -it -d --restart=always --name GSFM-notebook -p 12101:8888 -v `pwd`/project:/project gsfm-script; jupyter-notebook --ip=0.0.0.0 --port=8888 --allow-root --no-browser
```

## Evaluation signals

- brca_rs_gsfm_ranking.csv exists and contains a WYE-354 entry with a significant (top-ranked, strongly negative) RSGSFM, matching the paper's reported −0.92
- ROC-based 10-fold cross-validation and enrichment-based accuracyscore (r × (−log(P_Enrichment))) support retrieval performance at the P < 0.05 threshold used in the paper
- Effective compounds (IC50 < 10 μmol/L in ChEMBL v27 for MCF7) show more negative RSGSFM values than ineffective compounds (IC50 > 10 μmol/L), consistent with the reported positive RSGSFM–IC50 correlation
- Es are computed from ranked FM lists (esup, esdown) with RSGSFM = esup − esdown per Eqs. 12–16, so recomputation on the same matrices should reproduce the ranking

## Limitations

- The RS/RSGSFM method must be applied on GSFM-transformed matrices; retrieval performance depends on the BIQ combination and FM size chosen (50 cancer-related hallmark genesets used as FMs)
- Selecting functional modules from different sources can lead to slight variations in results, though disease-aligned modules remain robust
- Additional experimental and clinical studies are needed to validate and elucidate the mechanisms of identified candidates in each individual cancer

## Evidence

- [abstract] WYE-354 identification: "we identified WYE-354, perhexiline, and NTNCB as candidate therapeutic agents for the treatment of breast-invasive carcinoma"
- [intro] RSGSFM definition: "the genome-scale functional module-based reversal score (RSGSFM) was developed to compare FM pattern of drug perturbations with the disease-induced FM pattern"
- [methods] RS method provenance: "The RS method was adapted from the RGES developed in previous studies13"
- [methods] IC50 source: "Efficacy data for each compound (IC50) were sourced from the ChEMBL database (version 27)"
- [methods] Performance evaluation: "The area under the receiver operating characteristic curve (ROC)26 and the area under the precision-recall curve (PR)27 were utilized to assess the performance"
- [readme] Notebook execution: "run ./project/Notebooks BRCA_GSFM.ipynb"
