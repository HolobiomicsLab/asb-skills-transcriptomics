---
name: tf-target-enrichment-fisher-testing
description: Use when computing the network-level GSFM_TF column of a GSFM active
  matrix, i.e., when a gene expression matrix and a curated TF-target regulatory database
  (Garcia-Alonso et al.) must be combined to quantify FM activity at the transcriptional
  regulatory network level.
when_to_use_negative:
- When no TF-target regulatory database or TF expression data is available for the
  tumor type — GSFM_TF cannot be computed and only the gene-level (GSFM_Up, GSFM_Down)
  or pathway-level (GSFM_ssGSEA) quantifiers apply.
- When the goal is gene-level or pathway-level FM activity only — use GSFM_Up/GSFM_Down
  or GSFM_ssGSEA instead of the network-level GSFM_TF.
- When the input is already a transformed FM active matrix (e.g., output of STEP 2);
  this skill operates on the raw gene expression matrices.
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: GSFM
  role: Framework in which the GSFM_TF quantifier is computed and assembled into the
    FM active matrix; implemented in Python and R
  repo: https://github.com/LXY1201/GSFM
- name: R 4.1.1
  role: Runtime environment for the computational analyses including the Fisher exact
    enrichment tests
- name: Python 3.9.7
  role: Runtime environment for the GSFM implementation used in the data transformation
    step
- name: Docker
  role: Packages the gsfm-script image that runs the Jupyter notebook (e.g., BRCA_GSFM.ipynb)
    performing STEP 2 data transformation including GSFM_TF
- name: Jupyter Notebook
  role: Executes the BRCA_GSFM.ipynb notebook inside the GSFM-notebook container to
    compute the four BIQs
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.apsb.2024.08.017
    title: 'GSFM: A genome-scale functional module transformation to represent drug
      efficacy for in silico drug discovery'
  source_content_hash: sha256:bd1da702822d010791454fd8fafc5910bf509cc901bc8ac90b14213968871e5c
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
  - Python 3.9.7
  - R 4.1.1
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# TF-target enrichment Fisher testing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Identifies transcription factors (TFs) whose validated target genes are significantly enriched within each functional module (FM) via one-tailed Fisher exact tests, then computes the GSFM_TF quantifier as the regulation-weighted average expression of significantly regulated TFs. This supplies the transcriptional regulatory network-level dimension of the four biologically interpretable quantifiers (BIQs) in the GSFM transformation.

## When to use

Use when computing the network-level GSFM_TF column of a GSFM active matrix, i.e., when a gene expression matrix and a curated TF-target regulatory database (Garcia-Alonso et al.) must be combined to quantify FM activity at the transcriptional regulatory network level.

## When NOT to use

- When no TF-target regulatory database or TF expression data is available for the tumor type — GSFM_TF cannot be computed and only the gene-level (GSFM_Up, GSFM_Down) or pathway-level (GSFM_ssGSEA) quantifiers apply.
- When the goal is gene-level or pathway-level FM activity only — use GSFM_Up/GSFM_Down or GSFM_ssGSEA instead of the network-level GSFM_TF.
- When the input is already a transformed FM active matrix (e.g., output of STEP 2); this skill operates on the raw gene expression matrices.

## Inputs

- Processed cancer gene expression matrix (from STEP 1)
- Processed drug-induced gene expression matrix (from STEP 1)
- TF-target pairs database by Garcia-Alonso et al.
- 50 cancer-related Hallmark genesets defining the functional modules (FMs)

## Outputs

- GSFM_TF column per sample for each of the 50 FMs (transcriptional regulatory network-level activity)
- Cancer and drug-induced GSFM active matrices with the GSFM_TF quantifier assembled alongside GSFM_Up, GSFM_Down, and GSFM_ssGSEA

## How to apply

First filter the TF-target pairs from Garcia-Alonso et al. to high-confidence levels A and B, then retain only pairs with |r| > 0.2 and P < 0.05 in the tumor type under study. For each of the 50 cancer-related Hallmark genesets (FMs), perform a one-tailed Fisher exact test to identify TFs whose targets are significantly enriched within the module. Compute the regulation weight for each enriched TF as Ratio_Module x Ratio_TF, standardized so weights sum to 1 within each module. Finally calculate GSFM_TF as the weighted average of TF expression across significantly regulated TFs, producing one column per FM in the cancer and drug-induced GSFM active matrices.

## Related tools

- **GSFM** (Framework in which the GSFM_TF quantifier is computed and assembled into the FM active matrix; implemented in Python and R) — https://github.com/LXY1201/GSFM
- **R 4.1.1** (Runtime environment for the computational analyses including the Fisher exact enrichment tests)
- **Python 3.9.7** (Runtime environment for the GSFM implementation used in the data transformation step)
- **Docker** (Packages the gsfm-script image that runs the Jupyter notebook (e.g., BRCA_GSFM.ipynb) performing STEP 2 data transformation including GSFM_TF)
- **Jupyter Notebook** (Executes the BRCA_GSFM.ipynb notebook inside the GSFM-notebook container to compute the four BIQs)

## Examples

```
docker run -it -d --restart=always --name GSFM-notebook -p 12101:8888 --log-opt max-size=10m --log-opt max-file=5 -v `pwd`/project:/project gsfm-script
```

## Evaluation signals

- All TF-target pairs used must be restricted to Garcia-Alonso et al. confidence levels A and B with |r| > 0.2 and P < 0.05 in the tumor type.
- TF regulation weights (Ratio_Module x Ratio_TF) must sum to 1 within each module after standardization.
- The resulting FM active matrix must be complete: all 50 FMs covered for all samples with the GSFM_TF column present alongside the other three BIQ columns.
- Module activities should reproduce the expected mutual exclusivity pattern across modules described in the paper.
- GSFM_TF values should reflect weighted average TF expression and therefore fall within the expression range of the constituent TFs.

## Limitations

- Requires prior knowledge of functional modules and a tumor-type-specific TF-target regulatory database; defining modules accurately is challenging for new datasets without prior knowledge.
- The method may not be applicable to certain rare diseases when normal samples are unavailable.
- Selecting modules from different sources can lead to slight variations in results.

## Evidence
