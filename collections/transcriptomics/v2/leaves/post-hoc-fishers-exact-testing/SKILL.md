---
name: post-hoc-fishers-exact-testing
description: Use when you have tabulated control vs. perturbed cell counts per Louvain
  cluster (from any integration embedding or ICAT sslouvain assignments), a G-test
  has flagged clusters with significant treatment enrichment, and you need per-cluster
  direction of enrichment for a Fig 5C-style comparison.
when_to_use_negative:
- Cluster-level treatment enrichment has not yet been established by the G-test —
  Fisher's exact post hoc testing is only meaningful for clusters flagged as significant
- The question is differential gene expression or state identification itself — use
  ICAT's NCFS weighting and semi-supervised clustering rather than count-tabulation
  tests
- Data are not discrete cell-by-cluster assignment counts (e.g., continuous expression
  values), since Fisher's exact test requires a 2x2 contingency table
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: scanpy
  role: Provides sc.tl.louvain clustering of the three integrated embeddings and sc.pp.log1p
    normalization of the GSE164240 count matrix that produces the cluster assignments
    being tested
- name: ICAT (icat-sc)
  role: Produces the upstream sslouvain semi-supervised cluster assignments against
    which the integration-derived Fisher's exact enrichment patterns are compared
  repo: https://github.com/BradhamLab/icat
- name: Seurat
  role: One of the three integration workflows whose Louvain clusters are tested with
    G-test + Fisher's exact post hoc for treatment enrichment
- name: Scanorama
  role: One of the three integration workflows whose Louvain clusters are tested with
    G-test + Fisher's exact post hoc for treatment enrichment
- name: Harmony
  role: One of the three integration workflows whose Louvain clusters are tested with
    G-test + Fisher's exact post hoc for treatment enrichment
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad278
    title: 'ICAT: a novel algorithm to robustly identify cell states following perturbations
      in single-cell transcriptomes'
  source_content_hash: sha256:d822cb92b0f068db38d757a1574add04370cb2d5030becd7ea9b8955a73f01d1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BradhamLab/icat
  tools:
  - Harmony
  - ICAT (icat-sc)
  - Scanorama
  - Seurat
  - numpy
  - scanpy
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# post-hoc Fisher's exact testing

## Summary

Follow-up significance testing that resolves the direction (depletion vs. enrichment) of perturbed-cell clustering after a per-cluster G-test, used to show that ICAT — not Seurat, Scanorama, or Harmony — resolves the sm50/pks2 PMC responses to MK-886 in the GSE164240 dataset.

## When to use

Use when you have tabulated control vs. perturbed cell counts per Louvain cluster (from any integration embedding or ICAT sslouvain assignments), a G-test has flagged clusters with significant treatment enrichment, and you need per-cluster direction of enrichment for a Fig 5C-style comparison.

## When NOT to use

- Cluster-level treatment enrichment has not yet been established by the G-test — Fisher's exact post hoc testing is only meaningful for clusters flagged as significant
- The question is differential gene expression or state identification itself — use ICAT's NCFS weighting and semi-supervised clustering rather than count-tabulation tests
- Data are not discrete cell-by-cluster assignment counts (e.g., continuous expression values), since Fisher's exact test requires a 2x2 contingency table

## Inputs

- Control vs. perturbed cell counts per Louvain cluster for each integration method (Seurat, Scanorama, Harmony) on the processed GSE164240 PMC scRNA-seq AnnData object
- ICAT sslouvain cluster assignments (stored in the sslouvain column of the returned AnnData object) for the control-defined and MK-886-perturbed cells
- Per-cluster G-test significance results used to select clusters for post hoc testing

## Outputs

- Per-cluster Fisher's exact test p-values and odds ratios indicating depletion vs. enrichment of perturbed cells
- A comparison table of cluster enrichment/depletion patterns across ICAT, Seurat, Scanorama, and Harmony reproducing the Fig 5C comparison

## How to apply

First run Louvain clustering (sc.tl.louvain) separately on each integration method's embedding (Seurat, Scanorama, Harmony) plus the ICAT sslouvain assignments on the log1p-normalized AnnData. For every Louvain cluster, compute per-cluster treatment enrichment with a G-test on the control vs. perturbed cell count table. For clusters with significant G-test results, construct a 2x2 contingency table per cluster (cells in cluster vs. cells outside, control vs. perturbed) and run a post hoc Fisher's exact test to determine the direction of enrichment — depletion vs. enrichment of MK-886-perturbed cells. Finally, compare the integration-derived cluster enrichment patterns against the upstream ICAT sslouvain assignments to reproduce the Fig 5C comparison. The rationale is that enrichment significance alone cannot distinguish a perturbation-unique depleted cluster from an enriched one, which is exactly the contrast the paper uses to show 'only ICAT identifies a perturbation-unique cellular response.'

## Related tools

- **scanpy** (Provides sc.tl.louvain clustering of the three integrated embeddings and sc.pp.log1p normalization of the GSE164240 count matrix that produces the cluster assignments being tested)
- **ICAT (icat-sc)** (Produces the upstream sslouvain semi-supervised cluster assignments against which the integration-derived Fisher's exact enrichment patterns are compared) — https://github.com/BradhamLab/icat
- **Seurat** (One of the three integration workflows whose Louvain clusters are tested with G-test + Fisher's exact post hoc for treatment enrichment)
- **Scanorama** (One of the three integration workflows whose Louvain clusters are tested with G-test + Fisher's exact post hoc for treatment enrichment)
- **Harmony** (One of the three integration workflows whose Louvain clusters are tested with G-test + Fisher's exact post hoc for treatment enrichment)

## Examples

```
from scipy.stats import fisher_exact; odds, p = fisher_exact([[n_perturbed_in_cluster, n_control_in_cluster], [n_perturbed_out, n_control_out]])
```

## Evaluation signals

- The resulting table must show that none of the Seurat/Scanorama/Harmony clusterings resolves both a depleted and an enriched perturbation response cluster
- The ICAT sslouvain assignments must resolve both a depleted and an enriched perturbation-response cluster, matching the paper's finding that 'only ICAT identifies a perturbation-unique cellular response'
- Fisher's exact tests are applied only to clusters with significant G-test results, not to all clusters indiscriminately
- Direction calls (depletion vs. enrichment) come from the sign/direction of the 2x2 contingency, and the same 2x2 tabulation procedure must be applied uniformly across all four methods for a fair Fig 5C comparison

## Limitations

- Fisher's exact testing here is post hoc and conditional on G-test significance, so it inherits any sensitivity of the upstream Louvain clustering resolution
- The test evaluates one cluster at a time against all others, so it does not correct for the multiplicity of clusters tested per method
- The comparison is dataset-specific: the conclusion that only ICAT resolves both response directions was established on the PMC dataset (GSE164240) with MK-886 perturbation, and the manuscript pipeline was run on a high performance cluster with CentOS, not guaranteed to run out-of-the-box elsewhere

## Evidence

- [abstract] only ICAT identifies a perturbation-unique cellular response: "We empirically validate ICAT in a developmental model and find that only ICAT identifies a perturbation-unique cellular response."
- [intro] semi-supervised clustering preserves control labels: "perform semi-supervised clustering such that the originally identified control labels remain constant, but cells from experimental conditions are free to cluster with any other cells regardless of"
- [readme] sslouvain column of returned AnnData: "The final clustering is stored in the `sslouvain` column of the returned `AnnData` object."
- [intro] ICAT designed for perturbation heterogeneity: "ICAT is a tool developed to better identify cell states in scRNAseq experiments where perturbations or some other biologic heterogeneity is present, such as gene knock-outs."
