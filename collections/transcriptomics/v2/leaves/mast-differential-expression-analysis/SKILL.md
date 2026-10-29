---
name: mast-differential-expression-analysis
description: Use when you have an ICAT-processed AnnData with an ICAT-upweighted matrix
  (adata.obsm['X_icat']) and need to verify which weighted marker genes are differentially
  expressed between treatment groups.
when_to_use_negative:
- Input data has not been processed by ICAT and no weighted gene list exists — DE
  on unweighted genes does not test the marker-stability assumption
- The goal is de novo cell-state discovery rather than validating differential expression
  of a predefined gene set — use the ICAT clustering workflow instead
edam_operation: http://edamontology.org/operation_3565
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: MAST
  role: Fits hurdle models for differential-expression testing of ICAT-weighted marker
    genes between control and perturbed treatments
- name: scanpy
  role: Applies log1p normalization to the count matrix before MAST model fitting
  repo: https://github.com/BradhamLab/icat
- name: AnnData
  role: Data structure carrying the ICAT-upweighted matrix in adata.obsm['X_icat']
    and treatment labels in adata.obs
  repo: https://github.com/BradhamLab/icat
- name: ICAT
  role: Upstream method that produced the weighted gene list being tested for differential-expression
    stability
  repo: https://github.com/BradhamLab/icat
- name: numpy
  role: Numerical support for matrix operations on the weighted data
  repo: https://github.com/BradhamLab/icat
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad278
    title: 'ICAT: a novel algorithm to robustly identify cell states following perturbations
      in single-cell transcriptomes'
  source_content_hash: sha256:8778603395d20e30512db61378cf57a591e67b188e649032b1820529cee59983
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BradhamLab/icat
  tools:
  - AnnData
  - ICAT
  - MAST
  - numpy
  - scanpy
  edam_operation: http://edamontology.org/operation_3565
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# mast-differential-expression-analysis

## Summary

Fit MAST hurdle models to test ICAT-weighted marker genes for differential expression between control and perturbed (MK-886) treatments, testing the assumption that ICAT-selected marker genes remain stably expressed across conditions. Outputs a DE results table used to validate marker gene stability against Supplementary Fig. S12.

## When to use

Use when you have an ICAT-processed AnnData with an ICAT-upweighted matrix (adata.obsm['X_icat']) and need to verify which weighted marker genes are differentially expressed between treatment groups.

## When NOT to use

- Input data has not been processed by ICAT and no weighted gene list exists — DE on unweighted genes does not test the marker-stability assumption
- The goal is de novo cell-state discovery rather than validating differential expression of a predefined gene set — use the ICAT clustering workflow instead

## Inputs

- ICAT-processed AnnData (.h5ad) with control and MK-886-treated cells
- ICAT-upweighted count matrix stored in adata.obsm['X_icat']
- ICAT-weighted gene list produced by the upstream ICAT clustering task
- Treatment labels in adata.obs ('control' vs 'perturbed')

## Outputs

- mast_de_results.csv with columns: gene, log2FC, p-value, FDR, significance flag
- Set of differentially expressed marker genes (log2 fc > 1, FDR < 0.05) comparable to Supplementary Fig. S12

## How to apply

Load the ICAT-processed AnnData containing control and MK-886-treated cells, including the ICAT-upweighted count matrix in adata.obsm['X_icat'] and the ICAT-weighted gene list from the upstream clustering task. Confirm treatment labels ('control' vs 'perturbed') in adata.obs and apply log1p normalization with scanpy. Fit MAST hurdle models testing every gene in the ICAT-weighted gene list for differential expression between the two treatment groups. Apply the significance thresholds log2 fold change > 1 and FDR < 0.05 to flag differentially expressed genes. Write the complete DE results (gene, log2FC, p-value, FDR, significance flag) to mast_de_results.csv and compare the flagged genes against the four marker genes reported in Supplementary Fig. S12.

## Related tools

- **MAST** (Fits hurdle models for differential-expression testing of ICAT-weighted marker genes between control and perturbed treatments)
- **scanpy** (Applies log1p normalization to the count matrix before MAST model fitting) — https://github.com/BradhamLab/icat
- **AnnData** (Data structure carrying the ICAT-upweighted matrix in adata.obsm['X_icat'] and treatment labels in adata.obs) — https://github.com/BradhamLab/icat
- **ICAT** (Upstream method that produced the weighted gene list being tested for differential-expression stability) — https://github.com/BradhamLab/icat
- **numpy** (Numerical support for matrix operations on the weighted data) — https://github.com/BradhamLab/icat

## Examples

```
sc.pp.log1p(adata); model.cluster(adata, adata.obs['treatment']); X_icat = adata.obsm['X_icat']
```

## Evaluation signals

- All four reported marker genes show log2 fold change > 1 and FDR < 0.05 in the MAST results
- The four-gene DE outcome reproduces the marker-stability result shown in Supplementary Fig. S12
- mast_de_results.csv contains gene, log2FC, p-value, FDR, and significance-flag columns for every gene in the ICAT-weighted list
- Treatment labels in adata.obs are confirmed as exactly 'control' and 'perturbed' before model fitting
- Count matrix is log1p-normalized prior to hurdle-model fitting

## Limitations

- The skill tests only genes present in the ICAT-weighted gene list; differential expression of unweighted genes is not assessed
- Four marker genes were differentially expressed in the PMC dataset — these genes supported ICAT's identification of the verified sm50/pks2 perturbed cell states, so failure to recover them may indicate upstream ICAT weighting or label problems
- Results are specific to the control vs MK-886 contrast; other perturbation types may require re-tuned thresholds

## Evidence

- [results] MAST differential-expression analysis between treatments found that, importantly, four marker genes were considered differentially expressed in the PMC dataset (log2 fc > 1, FDR < 0.05; Supplementary Fig. S12): [evidence span withheld — anchor status: not_found]
- [methods] Load the ICAT-processed AnnData containing control and MK-886-treated cells: [evidence span withheld — anchor status: not_found]
- [readme] ICAT makes heavy use of the excellent scanpy library along with the associated AnnData data structure: "ICAT makes heavy use of the excellent `scanpy` library along with the associated `AnnData` data structure."
- [methods] Apply the significance thresholds and write results: [evidence span withheld — anchor status: not_found]
