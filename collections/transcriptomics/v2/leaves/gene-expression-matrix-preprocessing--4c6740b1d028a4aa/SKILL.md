---
name: gene-expression-matrix-preprocessing
description: 'Use when building gene expression inputs for the GSFM FM active matrix:
  raw tumor/normal matrices (TCGA, GEO) and LINCS drug-induced profiles must be sample-filtered
  for cell-line correlation before STEP 2 transformation.'
when_to_use_negative:
- Input is already the STEP-1 'Processing Cancer/Drug-induced Gene Expression Matrix'
  consumed by STEP 2 or STEP 3 — re-filtering would discard valid samples
- Goal is FM transformation (STEP 2) or reversal score calculation (STEP 3 RS-GSFM/RS-Gene/RS-Viper)
  rather than sample quality filtering of raw expression matrices
- Rare-disease data with no matched normal samples or relevant CCLE cell line for
  correlation-based filtering
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_2640
tools:
- name: GSFM
  role: Overall pipeline whose STEP 1 data-processing stage this skill implements;
    scripts at github.com/LXY1201/GSFM
  repo: https://github.com/LXY1201/GSFM
- name: signatureSearch
  role: R package 1.4.6 used to download LINCS drug-induced gene expression profiles
    for filtering
  repo: https://github.com/LXY1201/GSFM
- name: R
  role: R 4.1.1 environment (RStudio 4.1.1) in which the correlation filtering scripts
    run
  repo: https://github.com/LXY1201/GSFM
- name: Docker
  role: Hosts the GSFM-notebook container that runs the downstream notebooks consuming
    the STEP 1 outputs
  repo: https://github.com/LXY1201/GSFM
- name: Jupyter Notebook
  role: Runs BRCA_GSFM.ipynb (STEP 2) against the filtered matrices
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.apsb.2024.08.017
    title: 'GSFM: A genome-scale functional module transformation to represent drug
      efficacy for in silico drug discovery'
  source_content_hash: sha256:0f9862a331511375097410a9fc8eb570e2e30089b5e084c713a19a24aaaff203
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
  - Python
  - R
  - signatureSearch
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# gene-expression-matrix-preprocessing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

STEP 1 of the GSFM pipeline: quality-filter cancer gene expression matrices (TCGA BRCA/LUAD, CRPC GSE70768) and LINCS drug-induced profiles by correlating each sample against MCF7, A549, and PC3 reference cell lines and removing non-correlated samples before functional module transformation.

## When to use

Use when building gene expression inputs for the GSFM FM active matrix: raw tumor/normal matrices (TCGA, GEO) and LINCS drug-induced profiles must be sample-filtered for cell-line correlation before STEP 2 transformation.

## When NOT to use

- Input is already the STEP-1 'Processing Cancer/Drug-induced Gene Expression Matrix' consumed by STEP 2 or STEP 3 — re-filtering would discard valid samples
- Goal is FM transformation (STEP 2) or reversal score calculation (STEP 3 RS-GSFM/RS-Gene/RS-Viper) rather than sample quality filtering of raw expression matrices
- Rare-disease data with no matched normal samples or relevant CCLE cell line for correlation-based filtering

## Inputs

- Cancer Gene Expression Matrix (TCGA BRCA and LUAD tumor/adjacent-normal profiles)
- CRPC dataset GSE70768 from GEO
- Drug-induced Gene Expression Matrix (LINCS, 8140 compounds x 30 cell lines, 24 h, 10 umol/L, 12328 genes, via signatureSearch)
- CCLE cell line expression data (reference for cell line identity)

## Outputs

- Filtered cancer gene expression matrix (only samples correlated to MCF7, A549, or PC3)
- Filtered drug-induced gene expression matrix (same correlation filtering applied)
- STEP-1 outputs directly loadable by STEP 2 GSFM transformation notebooks (e.g. BRCA_GSFM.ipynb)

## How to apply

Download LINCS drug-induced gene expression profiles with the signatureSearch R package 1.4.6, restricted to the 8140 compounds x 30 cell line dataset (24 h treatment, 10 umol/L, 12328 genes). Extract tumor and adjacent-normal profiles for BRCA and LUAD from TCGA and load CRPC dataset GSE70768 from GEO (noting PC3 models castration-resistant rather than primary prostate cancer), plus CCLE expression data as the cell-line reference. Map cell lines between LINCS and ChEMBL/CCLE by name with manual inspection, focusing on MCF7, A549, and PC3, which have the most compounds with both efficacy (IC50 from ChEMBL v27) and expression data. Compute per-sample correlations of each cancer sample against its relevant cell line profile and remove samples not correlated to MCF7, A549, or PC3; apply the same filtering logic to the drug-induced profiles. Write the filtered cancer and drug-induced matrices to disk as STEP 1 outputs so they load directly into the STEP 2 notebooks (e.g. BRCA_GSFM.ipynb) running in the gsfm-script Docker container.

## Related tools

- **GSFM** (Overall pipeline whose STEP 1 data-processing stage this skill implements; scripts at github.com/LXY1201/GSFM) — https://github.com/LXY1201/GSFM
- **signatureSearch** (R package 1.4.6 used to download LINCS drug-induced gene expression profiles for filtering) — https://github.com/LXY1201/GSFM
- **R** (R 4.1.1 environment (RStudio 4.1.1) in which the correlation filtering scripts run) — https://github.com/LXY1201/GSFM
- **Docker** (Hosts the GSFM-notebook container that runs the downstream notebooks consuming the STEP 1 outputs) — https://github.com/LXY1201/GSFM
- **Jupyter Notebook** (Runs BRCA_GSFM.ipynb (STEP 2) against the filtered matrices)

## Examples

```
cd 2.data transformation/docker && docker build --tag gsfm-script:latest --file GSFM.dockerfile . && docker run -it -d --name GSFM-notebook -p 12101:8888 -v `pwd`/project:/project gsfm-script
```

## Evaluation signals

- Every retained cancer sample is correlated to its target cell line (MCF7, A549, or PC3); non-correlated samples are removed
- The same correlation filtering logic was applied to both the cancer and the drug-induced gene expression matrices
- Filtered matrices are directly loadable by the STEP 2 GSFM transformation notebooks (e.g. BRCA_GSFM.ipynb) inside the gsfm-script Docker image
- LINCS input matches the paper's restriction: 8140 compounds x 30 cell lines, 24 h treatment, 10 umol/L, 12328 genes
- Cell line mapping between LINCS and ChEMBL/CCLE retained MCF7, A549, and PC3, the lines with the most compounds having both efficacy and expression data

## Limitations

- PC3 models castration-resistant rather than primary prostate cancer, so TCGA prostate comparisons must use the CRPC dataset GSE70768 accordingly
- Correlation-based filtering relies on CCLE reference expression, so it is not applicable to rare diseases when normal samples or matched cell lines are unavailable
- Manual inspection is needed when mapping cell line names between LINCS and ChEMBL/CCLE
- Selecting modules or data from different sources can lead to slight variations in downstream results

## Evidence
