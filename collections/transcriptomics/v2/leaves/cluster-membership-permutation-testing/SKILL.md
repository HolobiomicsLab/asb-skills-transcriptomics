---
name: cluster-membership-permutation-testing
description: Use when a SINCERA clustering run (hierarchical average/ward linkage,
  consensus, or tight clustering) on z-scored cell type-selective genes needs a significance
  check before cell type labels are finalized.
when_to_use_negative:
- Cells have not yet been assigned to clusters — cluster.permutation.analysis operates
  on an existing cluster.assignment result, so run gene selection (cluster.geneSelection)
  and cluster.assignment first
- The expression matrix has not been z-score transformed and filtered for expression
  selectivity — significance testing on unnormalized or non-selective gene sets conflates
  clustering quality with preprocessing artifacts
- Goal is differential expression or gene signature analysis rather than judging cluster
  quality — SINCERA handles those as separate downstream steps after cell types are
  defined
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: SINCERA
  role: Provides the cluster.permutation.analysis function used to assess cluster
    significance via cluster membership permutation, applied to cluster.assignment
    output on the z-scored expression matrix
  repo: github.com/xu-lab/SINCERA
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1007/978-1-4939-7710-9_15
    title: Single-Cell Transcriptome Analysis Using SINCERA Pipeline
  source_content_hash: sha256:5f32675d3c4bdef3a58c50ef623c766dcf935fdc9ab5601032cd5c3bebe63845
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - SINCERA
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# cluster-membership-permutation-testing

## Summary

Assess the statistical significance of scRNA-Seq cell clusters produced by SINCERA's cluster.assignment step by permuting cluster memberships, so that a clustering scheme (e.g., average-linkage, ward, consensus, or tight clustering on z-scored cell type-selective genes) is only accepted when permutation analysis supports it.

## When to use

Use when a SINCERA clustering run (hierarchical average/ward linkage, consensus, or tight clustering) on z-scored cell type-selective genes needs a significance check before cell type labels are finalized.

## When NOT to use

- Cells have not yet been assigned to clusters — cluster.permutation.analysis operates on an existing cluster.assignment result, so run gene selection (cluster.geneSelection) and cluster.assignment first
- The expression matrix has not been z-score transformed and filtered for expression selectivity — significance testing on unnormalized or non-selective gene sets conflates clustering quality with preprocessing artifacts
- Goal is differential expression or gene signature analysis rather than judging cluster quality — SINCERA handles those as separate downstream steps after cell types are defined

## Inputs

- SINCERA R S4 object containing the preprocessed, z-score transformed expression matrix of cell type-selective genes
- Cluster labels from cluster.assignment for each of the four clustering conditions (average linkage, ward, consensus, tight clustering)

## Outputs

- Cluster significance assessment via cluster membership permutation for each clustering scheme
- Basis for the accepted cell partition written to cluster_assignments.csv (cell identifiers and assigned cluster labels per clustering condition)

## How to apply

After running cluster.assignment on the SINCERA R S4 object loaded with the preprocessed, z-score-transformed expression matrix of cell type-selective genes (chosen via cluster.geneSelection and specificity.thresholdSelection), run the cluster.permutation.analysis function on each candidate clustering scheme. The permutation procedure shuffles cluster memberships to estimate how likely the observed partition is under random assignment, providing a significance assessment per scheme. Because SINCERA deliberately lets you compare four clustering conditions (average-linkage hierarchical with Pearson correlation distance, ward linkage, consensus clustering, and tight clustering), run the permutation analysis on each and use it together with plotMarkers inspection of known marker gene expression to accept or reject each scheme. A clustering is accepted only when cluster membership permutation supports significance AND known marker genes show selective, non-overlapping expression across clusters; if marker patterns are scattered or overlapping, iterate with different parameter settings rather than accepting the partition.

## Related tools

- **SINCERA** (Provides the cluster.permutation.analysis function used to assess cluster significance via cluster membership permutation, applied to cluster.assignment output on the z-scored expression matrix) — github.com/xu-lab/SINCERA

## Examples

```
cluster.permutation.analysis(sincera.obj)
```

## Evaluation signals

- Cluster membership permutation analysis supports cluster significance for every accepted clustering condition
- plotMarkers shows known marker genes with selective (non-overlapping) expression across the defined clusters; scattered marker expression means the scheme must be rejected
- Significance checks were run on all four clustering conditions (average linkage, ward, consensus, tight clustering), not just one
- The final accepted partition is recorded in cluster_assignments.csv with cell identifiers and cluster labels per clustering condition
- Unstable or scattered-marker schemes trigger iteration with different clustering parameters rather than acceptance

## Limitations

- Permutation significance alone is insufficient — SINCERA requires corroborating marker selectivity (non-overlapping marker expression), so a significant partition with scattered markers still fails validation
- Cell clustering directly influences all downstream analysis (gene signatures, regulatory networks, driving forces), so an erroneously accepted clustering propagates errors into every subsequent SINCERA step
- Iteration is expected: if validation fails, different clustering.method parameter settings must be tried on the identical input matrix

## Evidence

- [methods] clustering significance workflow: "Cell clustering and cell type identification is a key step in the pipeline and directly influences all downstream analysis."
