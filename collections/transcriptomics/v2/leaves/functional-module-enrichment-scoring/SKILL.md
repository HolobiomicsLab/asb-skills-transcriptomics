---
name: functional-module-enrichment-scoring
description: Use when raw cancer or drug-induced gene expression matrices must be
  converted into functional module activity scores before computing reversal scores
  (e.g., RSGSFM) against disease signatures.
when_to_use_negative:
- The disease under study is a rare disease where normal/adjacent samples are unavailable,
  since GSFM relies on tumor-versus-normal signature construction
- Functional modules cannot be reasonably defined for the new dataset because no prior
  disease-related module knowledge is available
- The goal is gene-level differential expression analysis alone, not a functional-module
  or network-level representation of drug efficacy
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: GSFM
  role: Core framework implementing the four BIQs (GSFM_Up, GSFM_Down, GSFM_ssGSEA,
    GSFM_TF) that transform expression matrices into FM active matrices
  repo: https://github.com/LXY1201/GSFM
- name: signatureSearch 1.4.6
  role: R package used to download LINCS drug-induced gene expression profiles that
    serve as input to the transformation
- name: Docker
  role: Containers the GSFM notebook environment (gsfm-script image) in which the
    transformation is executed
- name: R 4.1.1
  role: Runtime for step 2 data transformation together with Python 3.9.7
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.apsb.2024.08.017
    title: 'GSFM: A genome-scale functional module transformation to represent drug
      efficacy for in silico drug discovery'
  source_content_hash: sha256:7885e83299237a22ee4e3d6b4335bd3d189245fb25f3a3a92098bba64415269f
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
  - R 4.1.1
  - signatureSearch
  - signatureSearch 1.4.6
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# functional-module-enrichment-scoring

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Transforms noisy gene expression matrices into genome-scale functional module (GSFM) active matrices using four biologically interpretable quantifiers (GSFM_Up, GSFM_Down, GSFM_ssGSEA, GSFM_TF), converting gene-level signals into multi-dimension FM activity scores for in silico drug discovery.

## When to use

Use when raw cancer or drug-induced gene expression matrices must be converted into functional module activity scores before computing reversal scores (e.g., RSGSFM) against disease signatures.

## When NOT to use

- The disease under study is a rare disease where normal/adjacent samples are unavailable, since GSFM relies on tumor-versus-normal signature construction
- Functional modules cannot be reasonably defined for the new dataset because no prior disease-related module knowledge is available
- The goal is gene-level differential expression analysis alone, not a functional-module or network-level representation of drug efficacy

## Inputs

- Processed cancer gene expression matrix (step 1 output, from TCGA or GEO e.g. GSE70768, GSE3744, GSE42568, GSE109169)
- Processed drug-induced gene expression matrix (step 1 output, from LINCS via signatureSearch)
- Hallmark genesets (50 cancer-related FMs)
- TF-target pairs database by Garcia-Alonso et al. (confidence levels A and B)

## Outputs

- Cancer GSFM active matrix (FM activity matrix per sample)
- Drug-induced GSFM active matrix (FM activity matrix per perturbation)
- Per-sample/per-drug FM activity scores under GSFM_Up, GSFM_Down, GSFM_ssGSEA, and GSFM_TF

## How to apply

First process the cancer and drug-induced gene expression matrices from TCGA/GEO and LINCS (downloaded via the signatureSearch R package 1.4.6), correlating each sample with CCLE cell lines and removing uncorrelated samples. Define functional modules using the 50 cancer-related hallmark genesets. Then compute the four BIQs: GSFM_Up and GSFM_Down evaluate FM activities at the gene level, GSFM_ssGSEA calculates FM activity per sample at the pathway level, and GSFM_TF assesses FM activity at the transcriptional regulatory network level using the average expression of TFs, restricted to high-confidence TF-target pairs (confidence levels A and B) from Garcia-Alonso et al. Run the transformation in the provided Docker image (gsfm-script) via Jupyter Notebook (e.g., BRCA_GSFM.ipynb), producing cancer and drug-induced GSFM active matrices that feed downstream RS-GSFM reversal-score calculations.

## Related tools

- **GSFM** (Core framework implementing the four BIQs (GSFM_Up, GSFM_Down, GSFM_ssGSEA, GSFM_TF) that transform expression matrices into FM active matrices) — https://github.com/LXY1201/GSFM
- **signatureSearch 1.4.6** (R package used to download LINCS drug-induced gene expression profiles that serve as input to the transformation)
- **Docker** (Containers the GSFM notebook environment (gsfm-script image) in which the transformation is executed)
- **R 4.1.1** (Runtime for step 2 data transformation together with Python 3.9.7)

## Examples

```
docker build --tag gsfm-script:latest --file GSFM.dockerfile . && docker run -it -d --name GSFM-notebook -p 12101:8888 -v `pwd`/project:/project gsfm-script && jupyter-notebook --ip=0.0.0.0 --port=8888 --allow-root --no-browser
```

## Evaluation signals

- The resulting FM active matrix has dimensions = number of samples/perturbations x number of hallmark FMs (50 cancer-related genesets), not genes
- All four BIQ columns (GSFM_Up, GSFM_Down, GSFM_ssGSEA, GSFM_TF) are present and populated in the active matrix
- GSFM_TF scores are computed only over high-confidence TF-target pairs (Garcia-Alonso levels A and B)
- Downstream retrieval benchmarking: the transformed matrices should outperform other matching methods (CMap, XSum, ZhangScore, RS) on ROC and precision-recall curve metrics and in accuracyscore
- Sample filtering is verified: only samples correlated to the matching CCLE cell line are retained before transformation

## Limitations

- Accurately defining functional modules poses challenges when applying GSFM to new datasets without prior knowledge
- Selecting modules from different sources can lead to slight variations in results, although well-aligned disease-related modules remain robust
- The method may not be applicable to certain rare diseases when normal samples are unavailable
- Noisy and potentially unreliable gene expression data must be pre-processed (sample-cell-line correlation filtering) before transformation

## Evidence
