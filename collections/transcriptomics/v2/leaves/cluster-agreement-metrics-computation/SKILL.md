---
name: cluster-agreement-metrics-computation
description: Use when you have two or more categorical cell-annotation columns over
  the same cells (e.g. scTriangulate labels vs. HLCA/CellRef label-transfer results)
  and need an external-agreement benchmark to compare integration strategies.
when_to_use_negative:
- Input is raw counts with no categorical annotation columns — the metrics require
  a definitive one-annotation-per-cell assignment matrix (binary X of shape I * N)
  from a prior scTriangulate or label-transfer run.
- The candidate and reference labels are defined on different cells (no shared base
  dataset), since Homogeneity/Completeness/V-Measure need cell-level label pairs.
- You need to integrate conflicting label sets rather than benchmark an existing one
  — that is the scTriangulate lazy_run integration step, not the metrics computation.
edam_operation: http://edamontology.org/operation_0346
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: scikit-learn
  role: Provides the Homogeneity, Completeness, and V-Measure implementations used
    to score agreement against HLCA and CellRef references
- name: scTriangulate
  role: Produces the reconciled decision-level labeling whose external agreement is
    benchmarked, via lazy_run with win_fraction_cutoff and add_metrics='tfidf5'
  repo: https://github.com/frankligy/scTriangulate
- name: scanpy 1.7.2
  role: Preprocessing/QC framework in which the annotations and UMAP coordinates are
    attached to AnnData before scoring
- name: ClusterEnsembles
  role: Generates the ensemble-clustering baseline labels used as the comparison arm
    in the agreement table
  repo: https://github.com/tsano430/ClusterEnsembles
- name: cellHarmony
  role: Two-step label transfer (MarkerFinder top-60 marker centroids, Pearson rho
    > 0.2, then base self-reclassification) that produces the query annotations scored
    against references
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-023-36016-y
    title: Decision level integration of unimodal and multimodal single cell data
      with scTriangulate
  source_content_hash: sha256:870412b052712d6d1a55cf6409a2ca203a8b1dd03e1fc87b59ebb80bfd64ee66
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/frankligy/scTriangulate
  tools:
  - ClusterEnsembles
  - cellHarmony
  - scTriangulate
  - scanpy
  - scanpy 1.7.2
  - scikit-learn
  edam_operation: http://edamontology.org/operation_0346
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# cluster-agreement-metrics-computation

## Summary

Quantify how well a reconciled cell-population labeling (e.g. a scTriangulate solution, or an ensemble-clustering baseline) agrees with independent reference annotations (Azimuth CellRef 1.0, HLCA) using Homogeneity, Completeness, and V-Measure (sklearn conventions), including subdivided dendritic-cell and macrophage populations.

## When to use

Use when you have two or more categorical cell-annotation columns over the same cells (e.g. scTriangulate labels vs. HLCA/CellRef label-transfer results) and need an external-agreement benchmark to compare integration strategies.

## When NOT to use

- Input is raw counts with no categorical annotation columns — the metrics require a definitive one-annotation-per-cell assignment matrix (binary X of shape I * N) from a prior scTriangulate or label-transfer run.
- The candidate and reference labels are defined on different cells (no shared base dataset), since Homogeneity/Completeness/V-Measure need cell-level label pairs.
- You need to integrate conflicting label sets rather than benchmark an existing one — that is the scTriangulate lazy_run integration step, not the metrics computation.

## Inputs

- AnnData (.h5ad) with scTriangulate reconciled labels and UMAP coordinates attached (add_annotation / add_umap)
- Reference label columns from HLCA (Azimuth human-lung-v2) and LungMAP CellRef 1.0 mapped onto the base cells
- Ensemble-clustering (ClusterEnsembles) labels over the same cells as the baseline
- Optional reference-centroid files (MarkerFinder top-60 markers, Pearson rho > 0.2) used by cellHarmony label transfer

## Outputs

- lung_atlas_agreement_metrics.csv table of Homogeneity, Completeness, and V-Measure for scTriangulate vs. HLCA and CellRef references
- Baseline comparison of the same three metrics for ensemble clustering

## How to apply

Attach every transferred annotation (from the four lung studies GSE161383, GSE171524, GSE136831, EGAS00001004344 mapped onto the Wang 9-donor snRNA-Seq base via cellHarmony's two-step strategy) plus UMAP coordinates to the base AnnData with scTriangulate.preprocessing.add_annotation and add_umap, normalize raw UMI counts to log CPTT, and run scTriangulate (0.9.0 with scanpy 1.7.2) using win_fraction_cutoff=0.4 and add_metrics='tfidf5'. Then compute Homogeneity, Completeness, and V-Measure of the scTriangulate assignment against the HLCA (Azimuth human-lung-v2) and LungMAP CellRef 1.0 reference assignments using scikit-learn's convention, taking care to include the subdivided dendritic-cell and macrophage (MNP-Verse) populations. Finally compute the same triple of metrics for an ensemble-clustering baseline (ClusterEnsembles) so the reported relationship — scTriangulate agreeing better with the independent atlases — can be confirmed. Rationale: decision-level integration should be judged by external agreement with references the integrator never saw, not by internal metrics.

## Related tools

- **scikit-learn** (Provides the Homogeneity, Completeness, and V-Measure implementations used to score agreement against HLCA and CellRef references)
- **scTriangulate** (Produces the reconciled decision-level labeling whose external agreement is benchmarked, via lazy_run with win_fraction_cutoff and add_metrics='tfidf5') — https://github.com/frankligy/scTriangulate
- **scanpy 1.7.2** (Preprocessing/QC framework in which the annotations and UMAP coordinates are attached to AnnData before scoring)
- **ClusterEnsembles** (Generates the ensemble-clustering baseline labels used as the comparison arm in the agreement table) — https://github.com/tsano430/ClusterEnsembles
- **cellHarmony** (Two-step label transfer (MarkerFinder top-60 marker centroids, Pearson rho > 0.2, then base self-reclassification) that produces the query annotations scored against references)

## Examples

```
from sklearn.metrics import homogeneity_completeness_v_measure; h, c, v = homogeneity_completeness_v_measure(adata.obs['sctri_final'], adata.obs['hlca_label']); df.to_csv('lung_atlas_agreement_metrics.csv')
```

## Evaluation signals

- Internal consistency check: each V-Measure must equal its harmonic-mean definition V = 2*h*c/(h+c) to numerical precision.
- Range check: all three metrics must lie in [0,1]; values outside this interval indicate mis-aligned label columns or empty clusters.
- Directional check: scTriangulate's Homogeneity/Completeness/V-Measure against Azimuth CellRef and HLCA should exceed the ensemble-clustering baseline, reproducing the reported relationship.
- Cell-identity check: the annotation columns being compared must have the same cell index (one definitive annotation per cell) with no NaN labels after add_annotation.
- Population completeness: the subdivided dendritic-cell and macrophage populations must be present in the reference labels before scoring, or the metrics will be biased toward coarse lineages.

## Limitations

- scTriangulate agreement metrics were tuned with programmatic defaults (e.g. win_fraction_cutoff) that may not be optimal for all study designs, so benchmark conclusions are setting-dependent.
- Population naming across references remains largely non-standardized, so agreement values depend on label-mapping decisions between atlases.
- Integrating cluster predictions from dozens of sources significantly increases runtime, which limits how many reference arms can be benchmarked per run.

## Evidence

- [results] agreement vs ensemble baseline: [evidence span withheld — anchor status: not_found]
- [other] V-Measure harmonic-mean invariant: [evidence span withheld — anchor status: not_found]
- [methods] tool versions for the run: "All the preprocessing and QC steps were conducted using scTriangulate version 0.9.0 and scanpy version 1.7.2."
- [methods] HLCA reference mapping: "The same two-step mapping strategy was applied to HLCA (https://app.azimuth.hubmapconsortium.org/app/human-lung-v2)"
- [methods] attaching labels to AnnData: "The obtained annotations and umap coordinates were added to the existing AnnData by scTriangulate.preprocessing.add_annotation and scTriangulate.preprocessing.add_umap functions"
- [readme] unstable-cluster pruning caveat: "scTriangulate is a Python package to mix-and-match conflicting clustering results in single cell analysis, and generate reconciled clustering solutions."
