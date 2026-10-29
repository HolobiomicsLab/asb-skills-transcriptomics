---
name: z-score-differential-expression-thresholding
description: Use when converting a processed cancer or drug-induced gene expression
  matrix into the gene-level (GSFM_Up, GSFM_Down) components of a GSFM active matrix,
  i.e. when per-sample ratios of significantly up- or down-regulated genes per functional
  module are needed.
when_to_use_negative:
- The goal is pathway-level or transcriptional regulatory network-level FM activity
  — use GSFM_ssGSEA or GSFM_TF instead, which capture those dimensions rather than
  gene-level up/down ratios
- Normal samples are unavailable and no reference distribution exists to define Z-scores
  (the paper notes the method may not apply to rare diseases without normal samples)
- Functional modules have not yet been defined for the dataset — FM selection (50
  cancer-related Hallmark genesets) must precede the gene-level quantification
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3526
tools:
- name: GSFM
  role: Framework implementing the Z-score-based gene-level quantifiers (GSFM_Up,
    GSFM_Down) as part of the four-BIQ transformation
  repo: https://github.com/LXY1201/GSFM
- name: R 4.1.1
  role: Computational environment in which the Z-score thresholding and FM active
    matrix assembly are conducted
- name: Docker
  role: Builds the gsfm-script container image that packages the GSFM transformation
    scripts
- name: Jupyter Notebook
  role: Runs BRCA_GSFM.ipynb, the notebook that executes STEP 2 data transformation
    including GSFM_Up/GSFM_Down computation
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.apsb.2024.08.017
    title: 'GSFM: A genome-scale functional module transformation to represent drug
      efficacy for in silico drug discovery'
  source_content_hash: sha256:b5d90ecf0ebe03ca2b81bb18f46a588c12d01c49bb0c253a84099d277b45e662
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
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3526
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# z-score-differential-expression-thresholding

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A gene-level differential expression quantification step within the GSFM transformation that converts per-sample gene expression Z-scores into the GSFM_Up and GSFM_Down columns of the FM active matrix, using the |Z| > 1.96 (p < 0.05) significance threshold to count highly and lowly expressed genes within each of the 50 cancer-related Hallmark functional modules.

## When to use

Use when converting a processed cancer or drug-induced gene expression matrix into the gene-level (GSFM_Up, GSFM_Down) components of a GSFM active matrix, i.e. when per-sample ratios of significantly up- or down-regulated genes per functional module are needed.

## When NOT to use

- The goal is pathway-level or transcriptional regulatory network-level FM activity — use GSFM_ssGSEA or GSFM_TF instead, which capture those dimensions rather than gene-level up/down ratios
- Normal samples are unavailable and no reference distribution exists to define Z-scores (the paper notes the method may not apply to rare diseases without normal samples)
- Functional modules have not yet been defined for the dataset — FM selection (50 cancer-related Hallmark genesets) must precede the gene-level quantification

## Inputs

- Processed cancer gene expression matrix (output of STEP 1 data processing)
- Processed drug-induced gene expression matrix (output of STEP 1 data processing)
- 50 cancer-related Hallmark genesets defining the functional modules (FMs)

## Outputs

- GSFM_Up column per FM per sample (ratio of genes with Z-score > 1.96 within the module)
- GSFM_Down column per FM per sample (ratio of genes with Z-score < −1.96 within the module)
- Gene-level component of the multi-level GSFM active matrix (50 FMs x samples)

## How to apply

Start from the STEP 1 processed cancer and drug-induced gene expression matrices and the 50 cancer-related Hallmark genesets defining the functional modules (FMs). For each sample, Z-score-normalize gene expression, then compute GSFM_Up as the ratio of genes within each FM whose Z-score exceeds 1.96 (highly expressed, ~95th percentile) and GSFM_Down as the ratio of genes whose Z-score falls below −1.96 (lowly expressed, ~5th percentile). These two quantifiers evaluate FM activity specifically at the gene level and are assembled together with GSFM_ssGSEA (pathway level) and GSFM_TF (transcriptional regulatory network level) into the complete multi-level GSFM active matrix via the BRCA_GSFM.ipynb notebook in the gsfm-script Docker container. The rationale is that converting raw expression into module-level significance ratios reduces noise and produces a more dependable FM active matrix than raw gene-level signals.

## Related tools

- **GSFM** (Framework implementing the Z-score-based gene-level quantifiers (GSFM_Up, GSFM_Down) as part of the four-BIQ transformation) — https://github.com/LXY1201/GSFM
- **R 4.1.1** (Computational environment in which the Z-score thresholding and FM active matrix assembly are conducted)
- **Docker** (Builds the gsfm-script container image that packages the GSFM transformation scripts)
- **Jupyter Notebook** (Runs BRCA_GSFM.ipynb, the notebook that executes STEP 2 data transformation including GSFM_Up/GSFM_Down computation)

## Examples

```
docker run -it -d --restart=always --name GSFM-notebook -p 12101:8888 -v `pwd`/project:/project gsfm-script; jupyter-notebook --ip=0.0.0.0 --port=8888 --allow-root --no-browser  # then run ./project/Notebooks BRCA_GSFM.ipynb
```

## Evaluation signals

- Completeness check: the resulting FM active matrix covers all 50 FMs x all samples with the four BIQ columns (including GSFM_Up and GSFM_Down) populated
- GSFM_Up and GSFM_Down values are ratios of genes within a module and therefore fall in [0, 1] per FM per sample
- Only genes with |Z-score| > 1.96 contribute to the numerator of the ratio, so perturbing the threshold should monotonically change GSFM_Up/GSFM_Down values
- Module activities in the assembled matrix show the expected mutual exclusivity pattern across modules described in the paper
- Re-running BRCA_GSFM.ipynb in the gsfm-script container reproduces the same FM active matrix from the same STEP 1 inputs

## Limitations

- The 1.96 Z-score cutoff assumes an approximately normal per-sample expression distribution; the ratio-based quantifier discards magnitude of differential expression
- Accurately defining functional modules is challenging when applying GSFM to new datasets without prior knowledge, and module selection from different sources leads to slight variations in results
- The method may not be applicable to certain rare diseases when normal samples are unavailable for Z-score computation

## Evidence
