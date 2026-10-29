---
name: categorical-treatment-enrichment-g-test
description: Use when you have categorical cluster assignments (Louvain or sslouvain)
  plus a binary treatment label (control vs perturbed) and need per-cluster enrichment/depletion
  statistics to judge whether a clustering resolves treatment responses.
when_to_use_negative:
- The cell grouping is already a continuous score or embedding coordinate rather than
  a categorical cluster label — build the contingency table only after clustering
- The treatment variable has more than two levels (e.g. multiple doses) without restructuring
  the contingency framework, since the per-cluster test here is 2x2 control-vs-perturbed
- You need to discover treatment-affected cell states de novo — this skill evaluates
  an existing clustering; state discovery itself is done upstream by ICAT's NCFS weighting
  and semi-supervised clustering
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: scanpy
  role: Provides Louvain clustering (sc.tl.louvain) on integrated embeddings and the
    AnnData structure holding the cell-by-cluster/treatment observations used for
    the contingency tables
- name: AnnData
  role: Data structure holding normalized counts, treatment labels, and cluster assignments
    (sslouvain, louvain) that feed the per-cluster count tables
- name: Seurat
  role: One of the integration workflows whose Louvain-clustered embedding is evaluated
    with the G-test/Fisher enrichment analysis
- name: Scanorama
  role: One of the integration workflows whose Louvain-clustered embedding is evaluated
    with the G-test/Fisher enrichment analysis
- name: Harmony
  role: One of the integration workflows whose Louvain-clustered embedding is evaluated
    with the G-test/Fisher enrichment analysis
- name: ICAT
  role: Upstream method producing the sslouvain assignments against which integration-method
    enrichment patterns are compared (Fig 5C)
  repo: https://github.com/BradhamLab/icat
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad278
    title: 'ICAT: a novel algorithm to robustly identify cell states following perturbations
      in single-cell transcriptomes'
  source_content_hash: sha256:5567edf84dee68a6f71608260ac765645c8992ef46e8941bcc0b48eb14ba193c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BradhamLab/icat
  tools:
  - AnnData
  - Harmony
  - ICAT
  - Scanorama
  - Seurat
  - numpy
  - scanpy
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# categorical-treatment-enrichment-g-test

## Summary

A per-cluster contingency-table test (G-test followed by post hoc Fisher's exact tests) that quantifies whether Louvain clusters from integrated scRNA-seq embeddings are depleted or enriched for perturbed (e.g. MK-886-treated) cells relative to control, used to compare ICAT sslouvain assignments against Seurat/Scanorama/Harmony clusterings.

## When to use

Use when you have categorical cluster assignments (Louvain or sslouvain) plus a binary treatment label (control vs perturbed) and need per-cluster enrichment/depletion statistics to judge whether a clustering resolves treatment responses.

## When NOT to use

- The cell grouping is already a continuous score or embedding coordinate rather than a categorical cluster label — build the contingency table only after clustering
- The treatment variable has more than two levels (e.g. multiple doses) without restructuring the contingency framework, since the per-cluster test here is 2x2 control-vs-perturbed
- You need to discover treatment-affected cell states de novo — this skill evaluates an existing clustering; state discovery itself is done upstream by ICAT's NCFS weighting and semi-supervised clustering

## Inputs

- AnnData object with log1p-normalized GSE164240 PMC counts and an obs column assigning each cell to a cluster (louvain or sslouvain)
- Binary treatment label per cell (e.g. obs['treatment'] = 'control' or 'perturbed'/MK-886)
- Cluster assignments from each integration method's embedding (Seurat, Scanorama, Harmony) plus ICAT sslouvain assignments

## Outputs

- Per-cluster 2x2 count tables of control vs. perturbed cells for each integration method
- Per-cluster G-test statistics and p-values for treatment enrichment
- Post hoc Fisher's exact test results indicating direction (depletion vs. enrichment of perturbed cells) for significant clusters
- Comparison table reproducing Fig 5C: integration-derived enrichment patterns vs. upstream ICAT sslouvain assignments

## How to apply

Tabulate control vs. perturbed cell counts for each cluster (e.g. Louvain clusters from Seurat, Scanorama, and Harmony embeddings, or ICAT sslouvain assignments) to form a 2xK contingency structure. For each cluster, compute per-cluster treatment enrichment with a G-test over the 2x2 table (that cluster vs. all others, control vs. perturbed). For clusters passing the G-test significance threshold, run post hoc Fisher's exact tests to determine the direction of the response — depletion vs. enrichment of perturbed cells. The method is validated by whether the clustering yields both a depleted and an enriched perturbation-response cluster: on the GSE164240 PMC dataset, ICAT's sslouvain assignments resolve both, whereas Seurat, Scanorama, and Harmony clusterings resolve neither, reproducing the Fig 5C comparison.

## Related tools

- **scanpy** (Provides Louvain clustering (sc.tl.louvain) on integrated embeddings and the AnnData structure holding the cell-by-cluster/treatment observations used for the contingency tables)
- **AnnData** (Data structure holding normalized counts, treatment labels, and cluster assignments (sslouvain, louvain) that feed the per-cluster count tables)
- **Seurat** (One of the integration workflows whose Louvain-clustered embedding is evaluated with the G-test/Fisher enrichment analysis)
- **Scanorama** (One of the integration workflows whose Louvain-clustered embedding is evaluated with the G-test/Fisher enrichment analysis)
- **Harmony** (One of the integration workflows whose Louvain-clustered embedding is evaluated with the G-test/Fisher enrichment analysis)
- **ICAT** (Upstream method producing the sslouvain assignments against which integration-method enrichment patterns are compared (Fig 5C)) — https://github.com/BradhamLab/icat

## Examples

```
import scanpy as sc; sc.tl.louvain(adata); tab = pd.crosstab(adata.obs['louvain'], adata.obs['treatment']); g, p, dof = power_divergence(tab.values, axis=0, lambda_='log-likelihood')
```

## Evaluation signals

- The G-test table must show that none of the Seurat/Scanorama/Harmony clusterings resolves both a depleted and an enriched perturbation-response cluster
- ICAT sslouvain assignments must show both a depleted and an enriched perturbation-response cluster, matching the paper's claim that 'only ICAT identifies a perturbation-unique cellular response'
- For every G-test-significant cluster, the post hoc Fisher's exact test must assign a definite direction (depletion or enrichment of perturbed cells)
- Per-cluster counts must sum to the total cell numbers of the control and perturbed samples for each integration method's clustering

## Limitations

- Validation is specific to the PMC dataset and MK-886 perturbation; the result that integration methods miss both sm50 and pks2 responses need not generalize to other datasets
- Louvain clustering resolution and integration parameters strongly affect cluster compositions, so the enrichment outcome is sensitive to upstream parameter choices
- The G-test per-cluster comparison tests treatment enrichment within existing clusters; it cannot recover treatment-specific states that integration workflows merge into mixed clusters

## Evidence

- [other] Tabulate control vs. perturbed cell counts per Louvain cluster for each integration method and compute per-cluster treatment enrichment with a G-test.: [evidence span withheld — anchor status: not_found]
- [abstract] only ICAT identifies a perturbation-unique cellular response: "We empirically validate ICAT in a developmental model and find that only ICAT identifies a perturbation-unique cellular response."
- [readme] ICAT uses sslouvain semi-supervised clustering with control labels fixed: "perform semi-supervised clustering such that the originally identified control labels remain constant, but cells from experimental conditions are free to cluster"
- [methods] install and run ICAT with scanpy/AnnData: "ICAT makes heavy use of the excellent `scanpy` library along with the associated `AnnData` data structure."
