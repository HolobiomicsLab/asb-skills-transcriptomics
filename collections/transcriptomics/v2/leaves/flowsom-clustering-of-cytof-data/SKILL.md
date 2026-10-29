---
name: flowsom-clustering-of-cytof-data
description: Use when multi-sample IMC/CyTOF protein expression matrices and a manual
  cell-type annotation are available and you need sample-integrated, annotation-concordant
  clustering to quantify batch-removal benefit.
when_to_use_negative:
- Input is a single-sample, single-condition dataset where batch/sample variability
  is not a confounder — plain FlowSOM without scMerge2 adjustment suffices
- Only an embedding (low-dimensional representation) is available and an adjusted
  expression matrix cannot be produced; embedding-only outputs limit interpretability
  and downstream sample-level analysis
- Downstream analysis requires joint use of multi-modal information — scMerge2 integration
  was performed only within each modality
edam_operation: http://edamontology.org/operation_3432
tools:
- name: FlowSOM
  role: Self-organizing-map clustering of the original and scMerge2-adjusted IMC expression
    matrices, run with the number of clusters fixed at 17
- name: scMerge2
  role: Produces the sample-integrated adjusted expression matrix (hierarchical integration,
    pseudo-bulk building, pseudo-replication) whose FlowSOM clustering achieves the
    higher ARI (0.58 vs 0.13)
  repo: https://github.com/SydneyBioX/scMerge
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-023-39923-2
    title: Atlas-scale single-cell multi-sample multi-condition data integration using
      scMerge2
  source_content_hash: sha256:da76be3eb277cbc46eb5a2fd1b6896f374333c2881bcca4d8a20bf6b2224cbfe
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SydneyBioX/scMerge
  tools:
  - FlowSOM
  - scMerge2
  edam_operation: http://edamontology.org/operation_3432
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# flowsom-clustering-of-cytof-data

## Summary

FlowSOM clustering of CyTOF/imaging mass cytometry protein expression, applied here to the COVID-19 IMC dataset (23 samples, 38 protein markers), so that scMerge2-adjusted data yields cluster assignments that concord strongly with manual cell-type annotation (ARI rising from 0.13 to 0.58).

## When to use

Use when multi-sample IMC/CyTOF protein expression matrices and a manual cell-type annotation are available and you need sample-integrated, annotation-concordant clustering to quantify batch-removal benefit.

## When NOT to use

- Input is a single-sample, single-condition dataset where batch/sample variability is not a confounder — plain FlowSOM without scMerge2 adjustment suffices
- Only an embedding (low-dimensional representation) is available and an adjusted expression matrix cannot be produced; embedding-only outputs limit interpretability and downstream sample-level analysis
- Downstream analysis requires joint use of multi-modal information — scMerge2 integration was performed only within each modality

## Inputs

- COVID-19 imaging mass cytometry (IMC) dataset: 23 samples, 38 protein markers
- Manual cell-type annotation per cell
- scMerge2-adjusted expression matrix produced by hierarchical integration with pseudo-bulk building and pseudo-replication

## Outputs

- FlowSOM cluster assignments (17 clusters) for original and adjusted matrices
- ARI comparison table (ari_comparison.csv): original ARI 0.13 vs adjusted ARI 0.58
- Marker-enrichment heatmaps per cluster
- CD4/CD8a/CD20 marker-specificity scatter plots

## How to apply

Load the COVID-19 imaging mass cytometry dataset (23 samples, 38 protein markers) with its manual cell-type annotation, then run scMerge2 using its hierarchical integration with pseudo-bulk building and pseudo-replication to produce an adjusted expression matrix across samples. Cluster both the original and the scMerge2-adjusted matrices with FlowSOM, explicitly setting the number of clusters to 17. Compute the adjusted Rand index (ARI) of each clustering against the manual annotation; the rationale is that scMerge2 removes dataset variability in imaging mass cytometry experiments, so the adjusted-data clustering should substantially outperform the original (ARI 0.13 vs 0.58). Confirm cell-type identity of clusters by generating marker-enrichment heatmaps per cluster and CD4/CD8a/CD20 marker-specificity scatter plots. Finally, tabulate the ARI comparison (ari_comparison.csv) and save the figure files.

## Related tools

- **FlowSOM** (Self-organizing-map clustering of the original and scMerge2-adjusted IMC expression matrices, run with the number of clusters fixed at 17)
- **scMerge2** (Produces the sample-integrated adjusted expression matrix (hierarchical integration, pseudo-bulk building, pseudo-replication) whose FlowSOM clustering achieves the higher ARI (0.58 vs 0.13)) — https://github.com/SydneyBioX/scMerge

## Examples

```
BiocManager::install("scMerge")
```

## Evaluation signals

- ARI against the manual cell-type annotation for the adjusted-data clustering must substantially exceed the original-data ARI (reproducing the increase from 0.13 to 0.58)
- FlowSOM must be run with exactly 17 clusters on both original and adjusted matrices for the ARI comparison to be valid
- Marker-enrichment heatmaps must show coherent per-cluster marker enrichment consistent with manual cell-type identities
- CD4/CD8a/CD20 marker-specificity scatter plots must confirm that clusters separate the expected marker-positive populations
- The scMerge2 run must use hierarchical integration with pseudo-bulk building and pseudo-replication, and output an adjusted expression matrix (not just an embedding)

## Limitations

- On the original unadjusted data, FlowSOM clustering agrees poorly with manual annotation (ARI ≈ 0.13); clustering without prior scMerge2 adjustment is unreliable for this dataset
- scMerge2 integration demonstrated in the paper was performed within each modality only; joint multi-modal use of information is deferred to future work
- Embedding-only integration outputs increase interpretability challenges for downstream features such as cell–cell interactions or pathway information; an adjusted expression matrix is preferred for this workflow

## Evidence

- [abstract] scMerge2 removes dataset variability in CyTOF, imaging mass cytometry and CITE-seq experiments: "we demonstrate that scMerge2 can remove dataset variability in CyTOF, imaging mass cytometry and CITE-seq experiments"
- [discussion] three innovations of scMerge2: "This was achieved via the use of three essential innovations with hierarchical integration, pseudo-bulk building to minimise processing demand, and pseudo-replication"
- [discussion] embedding-only output limitation: "an embedding output by itself increases interpretability challenges since a low-dimensional representation does not naturally lend itself to the development of interpretable features"
- [readme] scMerge package purpose (README): "`scMerge` is a R package for merging and normalising single-cell RNA-Seq datasets."
