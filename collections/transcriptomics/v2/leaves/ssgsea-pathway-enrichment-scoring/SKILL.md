---
name: ssgsea-pathway-enrichment-scoring
description: Use when a gene expression matrix (e.g., TCGA tumor or LINCS drug-induced
  profiles) must be transformed into per-sample FM activity scores at the pathway
  level as part of the GSFM four-quantifier (GSFM_Up/Down/ssGSEA/TF) active-matrix
  construction.
when_to_use_negative:
- The analysis only needs gene-level up/down regulation proportions, where GSFM_Up/GSFM_Down
  (Z-score |1.96| threshold) are sufficient and ssGSEA adds no information.
- Input is already a GSFM/FM active matrix — the ssGSEA quantifier operates on gene
  expression matrices, not on transformed module-level tables.
- No adequate gene set definitions are available for the disease under study; the
  paper notes that defining functional modules without prior knowledge is a key limitation.
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: GSFM
  role: Framework whose GSFM_ssGSEA quantifier implements the ECDF-based pathway-level
    activity scoring
  repo: https://github.com/LXY1201/GSFM
- name: Docker
  role: Builds and runs the gsfm-script image that packages the GSFM transformation
    notebook
- name: Jupyter Notebook
  role: Executes BRCA_GSFM.ipynb, which computes the four BIQs including GSFM_ssGSEA
- name: R 4.1.1
  role: Runtime environment for the GSFM analysis scripts alongside Python 3.9.7
- name: Python 3.9.7
  role: Implementation language of the GSFM transformation pipeline
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.apsb.2024.08.017
    title: 'GSFM: A genome-scale functional module transformation to represent drug
      efficacy for in silico drug discovery'
  source_content_hash: sha256:4aee92308cb143141c7753c47810484e35eb4800d9745ba7e890f31d69681464
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# ssgsea-pathway-enrichment-scoring

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes per-sample pathway-level activity of the 50 cancer-related Hallmark functional modules (FMs) via a GSFM_ssGSEA quantifier that compares the empirical cumulative distribution functions (ECDFs) of gene expression ranks inside versus outside each FM, contributing the pathway-level column of the GSFM active matrix.

## When to use

Use when a gene expression matrix (e.g., TCGA tumor or LINCS drug-induced profiles) must be transformed into per-sample FM activity scores at the pathway level as part of the GSFM four-quantifier (GSFM_Up/Down/ssGSEA/TF) active-matrix construction.

## When NOT to use

- The analysis only needs gene-level up/down regulation proportions, where GSFM_Up/GSFM_Down (Z-score |1.96| threshold) are sufficient and ssGSEA adds no information.
- Input is already a GSFM/FM active matrix — the ssGSEA quantifier operates on gene expression matrices, not on transformed module-level tables.
- No adequate gene set definitions are available for the disease under study; the paper notes that defining functional modules without prior knowledge is a key limitation.

## Inputs

- Processed cancer gene expression matrix (from STEP 1, e.g., TCGA BRCA/LUAD profiles)
- Processed drug-induced gene expression matrix (LINCS library, downloaded via signatureSearch R package)
- 50 cancer-related Hallmark genesets defining the functional modules (FMs)

## Outputs

- GSFM_ssGSEA pathway-level activity score per FM per sample
- GSFM active matrix (50 FMs x all samples x four BIQ columns) for cancer and drug-induced data

## How to apply

Take the processed cancer or drug-induced gene expression matrix from STEP 1 (sample-cell-line correlated, per the README) and the 50 cancer-related Hallmark genesets that define the FMs. Rank all genes within each sample, then compute the normalized difference between the ECDF of expression ranks for genes inside each FM and the ECDF for genes outside it, using an enrichment weight alpha of 1, as specified for GSFM_ssGSEA in the GSFM transformation. Run the computation via the gsfm-script Docker image (BRCA_GSFM.ipynb in Jupyter Notebook) so results are consistent with the other three biologically interpretable quantifiers. The output GSFM_ssGSEA column is combined with GSFM_Up, GSFM_Down, and GSFM_TF into the complete FM active matrix (50 FMs x all samples), which downstream feeds RS_GSFM drug-efficacy scoring. Validate that every FM and sample is covered by all four BIQ columns and that module activities reproduce the expected mutual exclusivity pattern across modules.

## Related tools

- **GSFM** (Framework whose GSFM_ssGSEA quantifier implements the ECDF-based pathway-level activity scoring) — https://github.com/LXY1201/GSFM
- **Docker** (Builds and runs the gsfm-script image that packages the GSFM transformation notebook)
- **Jupyter Notebook** (Executes BRCA_GSFM.ipynb, which computes the four BIQs including GSFM_ssGSEA)
- **R 4.1.1** (Runtime environment for the GSFM analysis scripts alongside Python 3.9.7)
- **Python 3.9.7** (Implementation language of the GSFM transformation pipeline)

## Examples

```
docker run -it -d --restart=always --name GSFM-notebook -p 12101:8888 -v `pwd`/project:/project gsfm-script; jupyter-notebook --ip=0.0.0.0 --port=8888 --allow-root --no-browser; run ./project/Notebooks BRCA_GSFM.ipynb
```

## Evaluation signals

- Resulting FM active matrix is complete: all 50 FMs x all samples covered by the four BIQ columns including GSFM_ssGSEA.
- Enrichment weight alpha is set to 1 and gene ranks are computed per sample (not across samples).
- Module activities show the expected mutual exclusivity pattern across modules described in the paper.
- GSFM_ssGSEA scores feed a downstream RS_GSFM that correlates positively with drug IC50 (drug efficacy) as reported.
- Docker/jupyter environment matches the documented reproducibility setup (R 4.1.1, Python 3.9.7, Ubuntu 20.04.6).

## Limitations

- Accurately defining functional modules is challenging when applying GSFM to new datasets without prior knowledge.
- Selecting modules from different sources can lead to slight variations in the results, though well-aligned disease-related modules mitigate this.
- The overall GSFM approach may not be applicable to certain rare diseases when normal samples are unavailable.

## Evidence
