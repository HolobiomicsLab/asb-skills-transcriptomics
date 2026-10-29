---
name: tf-regulatory-weight-normalization
description: Use when converting a gene expression matrix (e.g., TCGA tumor/normal
  or LINCS drug-induced profiles) into a GSFM active matrix and you need network-level
  FM activity via TF regulation, complementing GSFM_Up/Down and GSFM_ssGSEA.
when_to_use_negative:
- Normal samples are unavailable (e.g., certain rare diseases), since differential
  correlation-based TF-target filtering (|r| > 0.2, P < 0.05 between tumor and normal)
  cannot be performed
- The analysis goal is per-gene differential expression rather than multi-level functional
  module activity scoring
- The FMs under study have no significantly enriched TFs or are not well-aligned with
  the disease under study, making the TF-regulatory quantifier uninformative
edam_topics:
- http://edamontology.org/topic_0602
tools:
- name: GSFM
  role: Framework implementing the GSFM_TF quantifier and the FM active matrix transformation
  repo: https://github.com/LXY1201/GSFM
- name: Docker
  role: Runs the gsfm-script container hosting the BRCA_GSFM.ipynb Jupyter notebook
    that executes the TF weight computation
- name: Jupyter Notebook
  role: Interactive environment in which the GSFM_TF calculation notebook is run
- name: R 4.1.1
  role: Runtime environment for the data transformation scripts
- name: Python 3.9.7
  role: Runtime environment for the data transformation scripts
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.apsb.2024.08.017
    title: 'GSFM: A genome-scale functional module transformation to represent drug
      efficacy for in silico drug discovery'
  source_content_hash: sha256:3dd8cb28dcd87ccacce701e962ff49ce237aed44e25750040af2b88920e8158f
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
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# tf-regulatory-weight-normalization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes the GSFM_TF quantifier, a transcriptional regulatory network-level activity score for each functional module (FM), by weighting TF expression with normalized regulation weights (Ratio_Module x Ratio_TF, standardized to sum to 1 within each module) derived from high-confidence TF-target interactions.

## When to use

Use when converting a gene expression matrix (e.g., TCGA tumor/normal or LINCS drug-induced profiles) into a GSFM active matrix and you need network-level FM activity via TF regulation, complementing GSFM_Up/Down and GSFM_ssGSEA.

## When NOT to use

- Normal samples are unavailable (e.g., certain rare diseases), since differential correlation-based TF-target filtering (|r| > 0.2, P < 0.05 between tumor and normal) cannot be performed
- The analysis goal is per-gene differential expression rather than multi-level functional module activity scoring
- The FMs under study have no significantly enriched TFs or are not well-aligned with the disease under study, making the TF-regulatory quantifier uninformative

## Inputs

- Processed cancer and drug-induced gene expression matrices (STEP 1 output)
- TF-target pairs database from Garcia-Alonso et al. (confidence levels A and B)
- 50 cancer-related Hallmark genesets defining the functional modules (FMs)

## Outputs

- GSFM_TF column of the FM active matrix (50 FMs x all samples) for cancer and drug-induced data
- Per-module lists of significantly enriched TFs (one-tailed Fisher exact test)

## How to apply

First, filter TF-target pairs from the Garcia-Alonso et al. database to high-confidence levels A and B, retaining only pairs with |r| > 0.2 and P < 0.05 in the tumor type under study. Second, for each of the 50 cancer-related Hallmark FMs, run a one-tailed Fisher exact test to identify TFs whose targets are significantly enriched within the FM. Third, compute regulation weights as Ratio_Module x Ratio_TF, standardized so the weights sum to 1 within each module. Finally, calculate GSFM_TF as the weighted average of TF expression using these normalized weights across significantly regulated TFs, producing one GSFM_TF column per FM in the active matrix. This is executed within the gsfm-script Docker container via the BRCA_GSFM.ipynb notebook (STEP 2 of the GSFM workflow).

## Related tools

- **GSFM** (Framework implementing the GSFM_TF quantifier and the FM active matrix transformation) — https://github.com/LXY1201/GSFM
- **Docker** (Runs the gsfm-script container hosting the BRCA_GSFM.ipynb Jupyter notebook that executes the TF weight computation)
- **Jupyter Notebook** (Interactive environment in which the GSFM_TF calculation notebook is run)
- **R 4.1.1** (Runtime environment for the data transformation scripts)
- **Python 3.9.7** (Runtime environment for the data transformation scripts)

## Examples

```
docker build --tag gsfm-script:latest --file GSFM.dockerfile . ; docker exec -it GSFM-notebook bash -c 'jupyter-notebook --ip=0.0.0.0 --port=8888 --allow-root --no-browser' # run ./project/Notebooks/BRCA_GSFM.ipynb
```

## Evaluation signals

- Regulation weights (Ratio_Module x Ratio_TF) sum to 1 within each module after standardization
- Only TFs whose targets pass the one-tailed Fisher exact enrichment test in the FM contribute to GSFM_TF
- TF-target pairs are restricted to Garcia-Alonso confidence levels A/B with |r| > 0.2 and P < 0.05 in the tumor type
- The resulting FM active matrix is complete: all 50 FMs x all samples covered by the four BIQ columns including GSFM_TF
- Module activities reproduce the expected mutual exclusivity pattern across FMs described in the paper

## Limitations

- Accurately defining functional modules is challenging when applying GSFM to new datasets without prior knowledge
- Selecting modules from different sources can lead to slight variations in the results
- The method may not be applicable to rare diseases when normal samples are unavailable

## Evidence
