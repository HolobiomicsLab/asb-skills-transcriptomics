---
name: conflicting-annotation-construction
description: Use when benchmarking decision-level cluster integration on a simulated
  dataset with a known ground truth, or when constructing multiple partially matching
  annotation solutions to test whether a method can reconcile over- and under-clustered
  inputs.
when_to_use_negative:
- Input annotations already agree or are non-conflicting — there is nothing to mix-and-match
  and ensemble comparison is uninformative
- No simulated or curated ground truth exists against which Homogeneity/Completeness/V-Measure
  can be computed
- The goal is a production clustering solution rather than a method benchmark, where
  arbitrarily degraded (over-/under-clustered) inputs would be inappropriate
edam_operation: http://edamontology.org/operation_3633
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0091
tools:
- name: scTriangulate
  role: Receives the four conflicting annotation columns via the query argument and
    lazy_run reconciles them with default settings (Shapley-value-based, win_fraction_cutoff=0.25)
  repo: https://github.com/frankligy/scTriangulate
- name: Splatter
  role: Generates the simulated benchmark dataset with splatSimulateGroup providing
    ground-truth clusters c1-c5
- name: ClusterEnsembles
  role: Baseline ensemble clustering (HGPA, MCLA, HBGF) run with default options on
    the same conflicting annotations for comparison
  repo: https://github.com/tsano430/ClusterEnsembles
- name: scanpy
  role: Hosts the AnnData object holding adata.X, adata.obs annotations, and adata.obsm['X_umap']
    consumed by ScTriangulate
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-36016-y
    title: Decision level integration of unimodal and multimodal single cell data
      with scTriangulate
  source_content_hash: sha256:34b82efc4ba4079a18c3f240257c3a4d43db1b04c79ccc86992abac76049a481
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/frankligy/scTriangulate
  tools:
  - ClusterEnsembles
  - Splatter
  - scTriangulate
  - scanpy
  edam_operation: http://edamontology.org/operation_3633
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# conflicting-annotation-construction

## Summary

Constructs deliberately conflicting, partially overlapping cell-cluster annotations (e.g., Broad/Median/Fine/Excessive partitions derived from a Splatter-simulated ground truth) so that scTriangulate's mix-and-match integration can be benchmarked against consensus ensemble methods (HGPA, MCLA, HBGF).

## When to use

Use when benchmarking decision-level cluster integration on a simulated dataset with a known ground truth, or when constructing multiple partially matching annotation solutions to test whether a method can reconcile over- and under-clustered inputs.

## When NOT to use

- Input annotations already agree or are non-conflicting — there is nothing to mix-and-match and ensemble comparison is uninformative
- No simulated or curated ground truth exists against which Homogeneity/Completeness/V-Measure can be computed
- The goal is a production clustering solution rather than a method benchmark, where arbitrarily degraded (over-/under-clustered) inputs would be inappropriate

## Inputs

- Splatter-simulated count matrix with known ground-truth cluster labels (splatSimulateGroup, 5 groups c1-c5)
- Four conflicting annotation columns partially overlapping the ground truth (Broad, Median, Fine, Excessive)
- AnnData with normalized expression (adata.X), conflicting annotations in adata.obs, and adata.obsm['X_umap']

## Outputs

- Reconciled cell-to-cluster assignments from scTriangulate lazy_run
- Per-method Homogeneity/Completeness/V-Measure metrics (simulation_benchmark_metrics.csv) for scTriangulate, HGPA, MCLA, HBGF
- Number of clusters recovered by each method

## How to apply

First simulate single-cell data with Splatter's splatSimulateGroup (e.g., Splatter 1.16.1, batchCells=3000, 5 clusters c1-c5, group.prob=(0.23,0.15,0.15,0.23,0.24), de.prob=(0.15,0.15,0.15,0.2,0.4)). Then build four annotations that only partially overlap the ground truth: 'Broad' merging c1+c2+c3, 'Median' merging c2+c3, 'Fine' sub-dividing c5, and 'Excessive' sub-dividing both c2 and c3. Pass these conflicting annotation columns to scTriangulate's ScTriangulate query argument and run lazy_run (version 0.9.0, default settings including win_fraction_cutoff=0.25) to derive the reconciled assignment, and run ClusterEnsembles with HGPA, MCLA, and HBGF on the same annotations. Finally compute Homogeneity, Completeness, and V-Measure against the simulated truth using sklearn conventions (H(C), H(C|K), H(K|C), harmonic mean) and record the number of clusters each method recovers.

## Related tools

- **scTriangulate** (Receives the four conflicting annotation columns via the query argument and lazy_run reconciles them with default settings (Shapley-value-based, win_fraction_cutoff=0.25)) — https://github.com/frankligy/scTriangulate
- **Splatter** (Generates the simulated benchmark dataset with splatSimulateGroup providing ground-truth clusters c1-c5)
- **ClusterEnsembles** (Baseline ensemble clustering (HGPA, MCLA, HBGF) run with default options on the same conflicting annotations for comparison) — https://github.com/tsano430/ClusterEnsembles
- **scanpy** (Hosts the AnnData object holding adata.X, adata.obs annotations, and adata.obsm['X_umap'] consumed by ScTriangulate)

## Examples

```
sctri = ScTriangulate(dir='./output', adata=adata, query=['broad','median','fine','excessive']); sctri.lazy_run()
```

## Evaluation signals

- simulation_benchmark_metrics.csv contains Homogeneity/Completeness/V-Measure rows for all four methods (scTriangulate, HGPA, MCLA, HBGF)
- scTriangulate's V-Measure matches or exceeds the ensemble baselines, consistent with the paper's reported superiority
- scTriangulate recovers the 5 ground-truth clusters c1-c5 without adding artifacts from the sub-divided Fine/Excessive annotations
- Number of clusters recovered per method is recorded alongside the metrics for direct comparison
- Metrics are computed with sklearn conventions: H(C), H(C|K), H(K|C), and harmonic-mean V-Measure

## Limitations

- Programmatic default analytical settings (e.g., win_fraction_cutoff=0.25, abs_thresh=10) may not be optimal for all study designs
- Integration of cluster predictions from dozens of sources (resolutions, modalities, reference annotations) significantly increases runtime; over ~15 annotation sets scTriangulate substitutes Shapley selection with combined rank-based selection
- The benchmark validates recovery on a Splatter simulation; performance on real partially-matching annotations may differ from the simulated ground-truth setting

## Evidence

- [methods] lazy_run on four conflicting annotations: "We apply the scTriangulate lazy_run function in version 0.9.0 on these four conflicting annotation solutions to derive the final cluster assignment (default settings)."
- [methods] ClusterEnsembles baselines with default options: "using the Python package ClusterEnsembles with default options (https://github.com/tsano430/ClusterEnsembles)."
- [intro] Shapley-value weighting of annotation players: "We consider each set of cluster annotations as an individual player and leverage the Shapley value from coalitional game theory to determine the relative importance"
- [readme] README query-argument invocation pattern: "sctri = ScTriangulate(dir='./output',adata=adata,query=['sctri_rna_leiden_1','sctri_rna_leiden_2','sctri_rna_leiden_3'])"
- [methods] Pruning threshold on final-vs-original assignment fraction: "the final annotation assignments are pruned according to the fraction of final versus original cell-cluster assignment (user-defined) and re-classified using a nearest-neighbor"
