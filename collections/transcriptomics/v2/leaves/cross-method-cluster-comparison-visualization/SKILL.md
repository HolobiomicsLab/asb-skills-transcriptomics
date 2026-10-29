---
name: cross-method-cluster-comparison-visualization
description: Use when multiple integration or clustering methods have been run on
  the same control/perturbed single-cell dataset and you must determine which method's
  clusters recover treatment-responsive cell states (e.g. sm50/pks2 PMC responses
  to MK-886).
when_to_use_negative:
- You only have a single clustering method's output and no cross-method comparison
  is required
- The dataset has no control/perturbed treatment labels, so per-cluster treatment
  enrichment (G-test/Fisher's exact) cannot be computed
- The goal is to identify novel cell states rather than to benchmark whether a method's
  clusters capture known treatment responses
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scanpy
  role: Log1p normalization, Louvain clustering on each integrated embedding (sc.tl.louvain),
    and statistical tabulation of treatment counts per cluster
  repo: https://github.com/BradhamLab/icat
- name: AnnData
  role: Data structure holding count matrices, treatment labels, and cluster assignments
    (obs.louvain, obs.sslouvain) across all methods
  repo: https://github.com/BradhamLab/icat
- name: icat-sc
  role: Produces the reference sslouvain assignments via semi-supervised clustering
    with NCFS-upweighted features, against which integration methods are compared
  repo: https://github.com/BradhamLab/icat
- name: Seurat
  role: One of the benchmark integration workflows whose Louvain clusters are tested
    for treatment enrichment
- name: Scanorama
  role: One of the benchmark integration workflows whose Louvain clusters are tested
    for treatment enrichment
- name: Harmony
  role: One of the benchmark integration workflows whose Louvain clusters are tested
    for treatment enrichment
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad278
    title: 'ICAT: a novel algorithm to robustly identify cell states following perturbations
      in single-cell transcriptomes'
  source_content_hash: sha256:7fc982659aadd1fd2fb0013bd9db7fd627e0614314a3d2e523870fd3ceefe207
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BradhamLab/icat
  tools:
  - AnnData
  - Harmony
  - Scanorama
  - Seurat
  - icat-sc
  - numpy
  - scanpy
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# cross-method-cluster-comparison-visualization

## Summary

Compares cell-state groupings produced by different integration/clustering methods (ICAT sslouvain vs. Seurat, Scanorama, Harmony Louvain clusters) on perturbation scRNA-seq data, quantifying per-cluster treatment enrichment to show which method resolves perturbation-unique responses.

## When to use

Use when multiple integration or clustering methods have been run on the same control/perturbed single-cell dataset and you must determine which method's clusters recover treatment-responsive cell states (e.g. sm50/pks2 PMC responses to MK-886).

## When NOT to use

- You only have a single clustering method's output and no cross-method comparison is required
- The dataset has no control/perturbed treatment labels, so per-cluster treatment enrichment (G-test/Fisher's exact) cannot be computed
- The goal is to identify novel cell states rather than to benchmark whether a method's clusters capture known treatment responses

## Inputs

- Processed single-cell count matrices for control and perturbed conditions (e.g. GSE164240 PMC dataset) as AnnData objects, log1p-normalized with scanpy
- Integrated embeddings from Seurat, Scanorama, and Harmony run on the same dataset
- ICAT sslouvain cluster assignments (out.obs['sslouvain'] from models.icat().cluster())
- Treatment labels per cell (e.g. adata.obs['treatment'] with 'control'/'perturbed' values)

## Outputs

- Per-method tables of control vs. perturbed cell counts per Louvain cluster
- Per-cluster G-test results for treatment enrichment
- Post hoc Fisher's exact test results indicating depletion vs. enrichment of perturbed cells per significant cluster
- Fig 5C-style comparison table of integration-derived clusters vs. ICAT sslouvain assignments

## How to apply

Run each integration workflow (Seurat, Scanorama, Harmony) on the same processed count matrices (e.g. GSE164240 control and MK-886-perturbed PMC data log1p-normalized in scanpy/AnnData), then perform Louvain clustering separately on each integrated embedding with sc.tl.louvain. For each method, tabulate control vs. perturbed cell counts per Louvain cluster and compute per-cluster treatment enrichment with a G-test; for significant clusters, run post hoc Fisher's exact tests to determine direction (depletion vs. enrichment of perturbed cells). Compare these integration-derived enrichment patterns against the upstream ICAT sslouvain assignments (produced via models.icat().cluster() on the same AnnData) to reproduce the Fig 5C-style comparison. The rationale is that a method resolving the perturbation-unique response should show at least one cluster significantly depleted of perturbed cells and another significantly enriched, whereas methods that over-harmonize conditions will not.

## Related tools

- **scanpy** (Log1p normalization, Louvain clustering on each integrated embedding (sc.tl.louvain), and statistical tabulation of treatment counts per cluster) — https://github.com/BradhamLab/icat
- **AnnData** (Data structure holding count matrices, treatment labels, and cluster assignments (obs.louvain, obs.sslouvain) across all methods) — https://github.com/BradhamLab/icat
- **icat-sc** (Produces the reference sslouvain assignments via semi-supervised clustering with NCFS-upweighted features, against which integration methods are compared) — https://github.com/BradhamLab/icat
- **Seurat** (One of the benchmark integration workflows whose Louvain clusters are tested for treatment enrichment)
- **Scanorama** (One of the benchmark integration workflows whose Louvain clusters are tested for treatment enrichment)
- **Harmony** (One of the benchmark integration workflows whose Louvain clusters are tested for treatment enrichment)

## Examples

```
out = model.cluster(adata, adata.obs['treatment']); print(out.obs['sslouvain'].unique())
```

## Evaluation signals

- The ICAT sslouvain assignments must resolve both a perturbation-depleted and a perturbation-enriched cluster, while none of the Seurat/Scanorama/Harmony clusterings resolves both, matching the paper's claim that 'only ICAT identifies a perturbation-unique cellular response'
- Each integration method's per-cluster table must show significant G-test clusters followed by Fisher's exact tests giving an unambiguous direction (depletion vs. enrichment of perturbed cells)
- Cluster counts per method must sum to the total cell count in the combined AnnData object (no cells dropped between tabulations)
- The final comparison should align with the Fig 5C pattern from the ICAT manuscript for the PMC/MK-886 dataset
- ICAT output must contain the expected clustering column: out.obs['sslouvain'].unique() returns non-empty cluster labels

## Limitations

- Hyperparameters (n_neighbors, resolution for Louvain and semi-supervised Louvain; sigma and reg for NCFS) must be optimized per dataset; README recommends optimizing on control cells before running the full ICAT workflow
- The manuscript analysis was run on a high-performance cluster with CentOS using a conda environment (environment.yaml/spec-file.txt) and snakemake; it 'is not guaranteed to run on your system out-of-the-box'
- Comparison outcomes depend on the upstream integration embeddings; a method that over-harmonizes control and perturbed conditions will mask treatment-responsive clusters

## Evidence

- [abstract] We empirically validate ICAT in a developmental model and find that only ICAT identifies a perturbation-unique cellular response.: "We empirically validate ICAT in a developmental model and find that only ICAT identifies a perturbation-unique cellular response."
- [other] Tabulate control vs perturbed cell counts per Louvain cluster and compute enrichment with G-test: [evidence span withheld — anchor status: not_found]
- [other] Post hoc Fisher's exact tests determine enrichment direction: [evidence span withheld — anchor status: not_found]
- [readme] ICAT final clustering stored in sslouvain column of returned AnnData: "The final clustering is stored in the `sslouvain` column of the returned `AnnData` object."
- [methods] ICAT uses scanpy and AnnData: "ICAT makes heavy use of the excellent `scanpy` library along with the associated `AnnData` data structure."
