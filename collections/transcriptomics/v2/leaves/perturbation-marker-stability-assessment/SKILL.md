---
name: perturbation-marker-stability-assessment
description: Use when a completed ICAT run (ICAT-upweighted matrix in adata.obsm['X_icat'],
  ICAT-weighted gene list, control vs perturbed labels) must be checked for marker-gene
  differential expression between treatments.
when_to_use_negative:
- The AnnData lacks treatment labels in adata.obs or an ICAT-weighted gene list —
  DE must be restricted to ICAT-upweighted markers, not all genes
- The goal is cell-state discovery rather than validating marker stability — ICAT's
  semi-supervised clustering (sslouvain) is the appropriate skill then
- Data are bulk RNA or non-AnnData single-cell matrices not processed by ICAT, so
  MAST hurdle models on the upweighted space are not applicable
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: MAST
  role: Fits hurdle models testing each ICAT-weighted gene for differential expression
    between control and MK-886 treatment groups
- name: scanpy
  role: log1p normalization of the ICAT-upweighted count matrix prior to MAST model
    fitting
- name: AnnData
  role: Data structure holding treatment labels in adata.obs and the upweighted matrix
    in adata.obsm['X_icat']
- name: ICAT / icat-sc
  role: Upstream producer of the ICAT-weighted gene list and upweighted count matrix
    that constrain the DE test set
  repo: https://github.com/BradhamLab/icat
- name: numpy
  role: Numeric handling of count matrices and log2 fold-change computation
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad278
    title: 'ICAT: a novel algorithm to robustly identify cell states following perturbations
      in single-cell transcriptomes'
  source_content_hash: sha256:794f478139d615dfff922f16a236846e94268a27414a34131fb3af38b5b5c235
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BradhamLab/icat
  tools:
  - AnnData
  - ICAT / icat-sc
  - MAST
  - numpy
  - scanpy
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# perturbation-marker-stability-assessment

## Summary

Tests the latent assumption that marker genes selected by ICAT's NCFS up-weighting are stable across treatments by fitting MAST hurdle models on the ICAT-processed AnnData and flagging differentially expressed markers (log2 fc > 1, FDR < 0.05).

## When to use

Use when a completed ICAT run (ICAT-upweighted matrix in adata.obsm['X_icat'], ICAT-weighted gene list, control vs perturbed labels) must be checked for marker-gene differential expression between treatments.

## When NOT to use

- The AnnData lacks treatment labels in adata.obs or an ICAT-weighted gene list — DE must be restricted to ICAT-upweighted markers, not all genes
- The goal is cell-state discovery rather than validating marker stability — ICAT's semi-supervised clustering (sslouvain) is the appropriate skill then
- Data are bulk RNA or non-AnnData single-cell matrices not processed by ICAT, so MAST hurdle models on the upweighted space are not applicable

## Inputs

- ICAT-processed AnnData (.h5ad) with ICAT-upweighted counts in adata.obsm['X_icat']
- Treatment labels in adata.obs ('control' vs 'perturbed', e.g. MK-886-treated)
- ICAT-weighted (NCFS-upweighted) gene list from the upstream clustering task

## Outputs

- mast_de_results.csv with columns: gene, log2FC, p-value, FDR, significance flag
- Set of differentially expressed marker genes meeting log2 fc > 1 and FDR < 0.05, comparable to Supplementary Fig. S12

## How to apply

Load the ICAT-processed AnnData containing control and MK-886-treated cells, taking the ICAT-upweighted count matrix from adata.obsm['X_icat'] and the ICAT-weighted gene list produced by the upstream clustering task. Confirm treatment labels ('control' vs 'perturbed') in adata.obs and apply log1p normalization to the count matrix with scanpy, mirroring the paper's preprocessing step. Fit MAST hurdle models testing every gene in the ICAT-weighted gene list for differential expression between the control and MK-886 treatment groups. Apply the paper's significance thresholds — log2 fold change > 1 and FDR < 0.05 — to flag which markers are treatment-dependent. The rationale is to verify whether ICAT's NCFS-upweighted markers behave as stable discriminators or shift between conditions; in the PMC dataset four marker genes were differentially expressed, and these contributed to ICAT's sensitive identification of the verified sm50/pks2 perturbed cell states.

## Related tools

- **MAST** (Fits hurdle models testing each ICAT-weighted gene for differential expression between control and MK-886 treatment groups)
- **scanpy** (log1p normalization of the ICAT-upweighted count matrix prior to MAST model fitting)
- **AnnData** (Data structure holding treatment labels in adata.obs and the upweighted matrix in adata.obsm['X_icat'])
- **ICAT / icat-sc** (Upstream producer of the ICAT-weighted gene list and upweighted count matrix that constrain the DE test set) — https://github.com/BradhamLab/icat
- **numpy** (Numeric handling of count matrices and log2 fold-change computation)

## Examples

```
model = models.icat(ctrl_value="control", ncfs_kws={'reg': 1, 'sigma': 3}); out = model.cluster(adata, adata.obs['treatment']); sc.pp.log1p(adata)
```

## Evaluation signals

- All four reported marker genes show log2 fold change > 1 and FDR < 0.05 in the MAST results, reproducing the outcome in Supplementary Fig. S12
- Exactly four ICAT-selected genes are flagged differentially expressed in the PMC dataset
- mast_de_results.csv has the expected schema: gene, log2FC, p-value, FDR, significance flag
- Every gene in the ICAT-weighted gene list has a fitted MAST hurdle model (no rows missing from the DE table)
- Treatment groups in adata.obs are limited to 'control' vs 'perturbed' before model fitting

## Limitations

- The paper's discussion text was not extractable, so nuances of the marker-stability interpretation are not documented in the available material
- Thresholds (log2 fc > 1, FDR < 0.05) and the four-gene result are specific to the MK-886 PMC dataset and Supplementary Fig. S12; they are not universal defaults
- The ICAT_manuscript repo states reproduction was run on a CentOS HPC and 'It is not guaranteed to run on your system out-of-the-box'

## Evidence

- [supplementary] MAST differential-expression analysis between treatments found that, importantly, four marker genes were considered differentially expressed in the PMC dataset (log2 fc > 1, FDR < 0.05: [evidence span withheld — anchor status: not_found]
- [results] these genes contributed to ICAT's ability to sensitively identify the verified sm50/pks2 perturbed cell states: [evidence span withheld — anchor status: not_found]
- [methods] log1p normalization of the AnnData prior to downstream analysis: "adata = controls.concatenate([perturbed])
    sc.pp.log1p(adata)"
- [readme] the ICAT-upweighted count matrix is stored in obsm['X_icat'] for downstream use such as visualization: "it is possible to pass the upweighted count matrix
(found in `adata.obsm["X_icat"]`) to these algorithms"
