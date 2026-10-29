---
name: cross-boundary-direction-correctness-evaluation
description: Use when comparing RNA velocity methods (e.g., UniTVelo vs scVelo stochastic/dynamical)
  on a dataset with known cell-type annotations and expected lineage transitions,
  to score direction correctness across cluster boundaries.
when_to_use_negative:
- The dataset lacks reliable ground-truth cell-type annotations or known lineage ordering,
  since cluster_edges must be supplied externally
- You need unsupervised trajectory quality assessment without prior biological transition
  expectations — the article notes selecting dynamically informed genes unsupervised
  remains an open challenge and this metric is supervision-dependent
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: UniTVelo
  role: Provides utv.evaluate to compute Cross Boundary Direction Correctness on velocity-inferred
    transitions
  repo: github.com/StatBiomed/UniTVelo
- name: scVelo
  role: Comparator baseline run in stochastic and dynamical modes whose velocities
    are scored with the same CBDir metric
provenance:
  source_task_ids:
  - task_002
  - task_004
  - task_007
  source_papers:
  - doi: 10.1038/s41467-022-34188-7
    title: 'UniTVelo: temporally unified RNA velocity reinforces single-cell trajectory
      inference'
  source_content_hash: sha256:a86d10764b935d66c8884875f33c08db834c4a42c97438d9cdfc38ef59c25f12
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python 3
  - UniTVelo
  - scVelo
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# cross-boundary-direction-correctness-evaluation

## Summary

Quantifies RNA velocity method accuracy using Cross-Boundary Direction Correctness (CBDir), the metric used in UniTVelo's Table 1 comparison of UniTVelo vs scVelo across ten datasets, by scoring whether inferred velocities point in the correct direction across ground-truth cell-type transition edges.

## When to use

Use when comparing RNA velocity methods (e.g., UniTVelo vs scVelo stochastic/dynamical) on a dataset with known cell-type annotations and expected lineage transitions, to score direction correctness across cluster boundaries.

## When NOT to use

- The dataset lacks reliable ground-truth cell-type annotations or known lineage ordering, since cluster_edges must be supplied externally
- You need unsupervised trajectory quality assessment without prior biological transition expectations — the article notes selecting dynamically informed genes unsupervised remains an open challenge and this metric is supervision-dependent

## Inputs

- AnnData object (h5ad) with spliced/unspliced count matrices and a fitted 'velocity' layer
- Cell-type annotation column name in adata.obs (label)
- Ground-truth cluster_edges (list of ordered cell-type transition tuples)

## Outputs

- metrics dict from utv.evaluate containing Cross Boundary Direction Correctness (CBDir) score
- Aggregated method x dataset CBDir comparison table (e.g., table1_cbdc_comparison.csv)

## How to apply

After fitting velocity with UniTVelo (utv.run_model) or scVelo on the same AnnData containing spliced/unspliced counts and a cell-type label column in adata.obs, define ground-truth cluster_edges tuples for the expected transitions (e.g., OPC→COP→NFOL→MFOL in the P20 mouse hindbrain pons, or erythroid/monocyte branches in human bone marrow). Call utv.evaluate(adata, cluster_edges, label, 'velocity'), which returns a metrics dict including the Cross Boundary Direction Correctness score. Repeat per method–dataset pair and aggregate scores into a comparison table (e.g., Table 1: mouse erythroid CBDir 0.793 for UniTVelo vs −0.445 dynamical and 0.129 stochastic scVelo; hindbrain 0.609 vs 0.347 and 0.074). Validate by checking the relative ranking matches the published table and that velocity stream embeddings visually support the same directionality.

## Related tools

- **UniTVelo** (Provides utv.evaluate to compute Cross Boundary Direction Correctness on velocity-inferred transitions) — github.com/StatBiomed/UniTVelo
- **scVelo** (Comparator baseline run in stochastic and dynamical modes whose velocities are scored with the same CBDir metric)

## Examples

```
import unitvelo as utv; metrics = utv.evaluate(adata, cluster_edges, label, 'velocity')
```

## Evaluation signals

- CBDir row exists for every method x dataset combination (UniTVelo, scVelo stochastic, scVelo dynamical)
- UniTVelo CBDir values reproduce the relative ranking of Table 1 (e.g., mouse erythroid 0.793 > 0.129 > −0.445)
- Velocity embedding streams (scv.pl.velocity_embedding_stream, dpi=100, colored by label) show directionality concordant with the CBDir score
- cluster_edges supplied to utv.evaluate match the biologically expected transitions for the dataset

## Limitations

- CBDir requires externally supplied ground truth via cluster_edges; it cannot be computed without prior knowledge of progenitor-to-differentiated ordering
- Possible over-counting in unspliced read counting may affect cell transition metrics derived from the RNA velocity vector
- Scores are dataset- and edge-definition-dependent; a low scVelo score (e.g., −0.445) reflects reversed direction on specific branches rather than overall failure

## Evidence

- [methods] Evaluation metrics via utv.evaluate: "# 4. Evaluation metrics (Optional):: # Cross Boundary Direction Correctness"
- [readme] README evaluation block: "metrics = utv.evaluate(adata, cluster_edges, label, 'velocity')"
- [results] Mouse erythroid Table 1 result: [evidence span withheld — anchor status: not_found]
- [results] Hindbrain CBDir comparison: [evidence span withheld — anchor status: not_found]
