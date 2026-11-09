---
name: icat-upweighted-gene-space-analysis
description: Use when you have an ICAT-processed AnnData (obsm['X_icat'] upweighted
  matrix, ICAT gene list, 'control' vs 'perturbed' treatment labels) and need to test
  whether ICAT-selected marker genes are differentially expressed between treatments.
when_to_use_negative:
- You have raw, unclustered single-cell counts with no ICAT weighting (no obsm['X_icat']
  or NCFS gene list) — run the upstream ICAT clustering/weighting workflow first
- Your goal is de novo differential-expression discovery across all genes rather than
  validating stability of a pre-selected ICAT marker set
- The AnnData lacks treatment labels or contains only one condition, so control-vs-perturbed
  MAST contrast is impossible
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: ICAT
  role: Produces the upweighted count matrix (adata.obsm['X_icat']) and NCFS-weighted
    gene list that restrict the MAST differential-expression testing space
  repo: https://github.com/BradhamLab/icat
- name: MAST
  role: Fits hurdle models testing every ICAT-weighted gene for differential expression
    between control and MK-886 treatment groups
- name: scanpy
  role: Performs log1p normalization (sc.pp.log1p) on the AnnData count matrix before
    fitting MAST models
- name: AnnData
  role: Data structure carrying the ICAT-upweighted matrix, treatment labels, and
    weighted gene list consumed by the MAST workflow
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad278
    title: 'ICAT: a novel algorithm to robustly identify cell states following perturbations
      in single-cell transcriptomes'
  source_content_hash: sha256:c4a67730c08cb73616bdce90c16bf7ffed14bded51a46146dc8798faec93ea82
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
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# icat-upweighted-gene-space-analysis

## Summary

Differential-expression stability testing of ICAT-weighted marker genes using MAST hurdle models, as in the ICAT paper's PMC (primary mesenchyme cell) analysis where control cells were compared against MK-886-treated cells to test the paper's latent assumption that ICAT-selected marker genes remain stable across treatment.

## When to use

Use when you have an ICAT-processed AnnData (obsm['X_icat'] upweighted matrix, ICAT gene list, 'control' vs 'perturbed' treatment labels) and need to test whether ICAT-selected marker genes are differentially expressed between treatments.

## When NOT to use

- You have raw, unclustered single-cell counts with no ICAT weighting (no obsm['X_icat'] or NCFS gene list) — run the upstream ICAT clustering/weighting workflow first
- Your goal is de novo differential-expression discovery across all genes rather than validating stability of a pre-selected ICAT marker set
- The AnnData lacks treatment labels or contains only one condition, so control-vs-perturbed MAST contrast is impossible

## Inputs

- ICAT-processed AnnData object (.h5ad) with ICAT-upweighted count matrix in adata.obsm['X_icat']
- ICAT-weighted gene list from the upstream ICAT clustering task (NCFS-weighted marker genes)
- adata.obs treatment labels ('control' vs 'perturbed', e.g. control vs MK-886-treated PMC cells)

## Outputs

- mast_de_results.csv with columns: gene, log2FC, p-value, FDR, significance flag
- Set of differentially expressed ICAT marker genes (log2FC > 1, FDR < 0.05; four genes in the PMC/MK-886 dataset)

## How to apply

Load the ICAT-processed AnnData containing control and MK-886-treated cells, using the ICAT-upweighted count matrix in adata.obsm['X_icat'] and the ICAT-weighted gene list produced by the upstream clustering task (ICAT first identifies control-defined cell states by unsupervised Louvain clustering, then weights predictive genes with NCFS). Confirm the treatment labels ('control' vs 'perturbed') in adata.obs and apply log1p normalization with scanpy, exactly as in the paper's methods. Fit MAST hurdle models testing every gene in the ICAT-weighted gene list for differential expression between the control and MK-886 groups. Apply the paper's significance thresholds — log2 fold change > 1 and FDR < 0.05 — to flag the four reported marker genes that were differentially expressed in the PMC dataset. Write the full results table (gene, log2FC, p-value, FDR, significance flag) to mast_de_results.csv and compare the four-gene outcome against Supplementary Fig. S12.

## Related tools

- **ICAT** (Produces the upweighted count matrix (adata.obsm['X_icat']) and NCFS-weighted gene list that restrict the MAST differential-expression testing space) — https://github.com/BradhamLab/icat
- **MAST** (Fits hurdle models testing every ICAT-weighted gene for differential expression between control and MK-886 treatment groups)
- **scanpy** (Performs log1p normalization (sc.pp.log1p) on the AnnData count matrix before fitting MAST models)
- **AnnData** (Data structure carrying the ICAT-upweighted matrix, treatment labels, and weighted gene list consumed by the MAST workflow)

## Examples

```
sc.pp.log1p(adata)  # normalize ICAT-processed AnnData, then fit MAST hurdle models on the ICAT-weighted gene list
```

## Evaluation signals

- All four reported marker genes show log2 fold change > 1 and FDR < 0.05 in the MAST results, reproducing Supplementary Fig. S12
- mast_de_results.csv contains the required columns (gene, log2FC, p-value, FDR, significance flag) and covers every gene in the ICAT-weighted gene list
- Only genes from the ICAT-weighted gene list are tested — the test space is the upweighted gene set, not the full transcriptome
- Treatment labels in adata.obs resolve to exactly two groups ('control' and 'perturbed') before model fitting

## Limitations

- Requires completion of the upstream ICAT clustering/NCFS-weighting task; without the X_icat matrix and weighted gene list this analysis cannot be run
- The DE outcome is dataset-dependent: in the PMC dataset only four ICAT marker genes passed the log2FC > 1, FDR < 0.05 thresholds, so most ICAT-weighted genes are expected to be stable
- The article's discussion section contained no extractable text, so caveats beyond the methods and README are not documented

## Evidence

- [methods] ICAT makes heavy use of the excellent `scanpy` library along with the associated `AnnData` data structure.: "ICAT makes heavy use of the excellent `scanpy` library along with the associated `AnnData` data structure."
- [intro] Apply sparse gene weighting via Neighborhood Component Feature Selection (NCFS) to weight predictive genes and down-weight non-explanatory genes: "These identified cell states are then fed into a sparse gene weighting algorithm, Neighborhood Component Feature Selection (NCFS), to highly weight the most predictive genes"
- [readme] it is possible to pass the upweighted count matrix (found in `adata.obsm["X_icat"]`) to these algorithms: "it is possible to pass the upweighted count matrix
(found in `adata.obsm["X_icat"]`) to these algorithms"
- [methods] ICAT can be installed on linux machines using `pip` with the following command: `pip install icat-sc`: "ICAT can be installed on linux machines using `pip` with the following command: `pip install icat-sc`"
