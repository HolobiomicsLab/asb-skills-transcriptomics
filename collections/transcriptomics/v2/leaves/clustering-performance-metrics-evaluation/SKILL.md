---
name: clustering-performance-metrics-evaluation
description: Use when a multi-clustering integration result (scTriangulate clusters)
  must be quantitatively compared against silver-standard references or individual
  modality/resolution clustering solutions using V-Measure, Homogeneity, and Completeness.
when_to_use_negative:
- You only have a single clustering solution and no reference labels to compare against
  — internal stability scores (reassign, TFIDF5, SCCAF) inside scTriangulate are the
  appropriate tools instead
- The task is cluster discovery or marker interpretation (e.g., identifying classical/intermediate/inflammatory
  CD14 monocytes) rather than quantitative agreement evaluation
tools:
- name: ClusterEnsembles
  role: Python package used with default options for computing clustering agreement
    metrics between predictions
  repo: https://github.com/tsano430/ClusterEnsembles
- name: scTriangulate 0.9.0
  role: Produces the integrated cell-population assignments whose V-Measure/Completeness
    are benchmarked against individual clustering solutions
  repo: https://github.com/frankligy/scTriangulate
- name: scanpy 1.7.2
  role: Provides the Leiden clustering solutions and preprocessing pipeline whose
    outputs feed the metric evaluation
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-023-36016-y
    title: Decision level integration of unimodal and multimodal single cell data
      with scTriangulate
  source_content_hash: sha256:f1aef686a980c5403420135ce90916575450b2fadc7e2531bccc707afdc10c2b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tsano430/ClusterEnsembles
  tools:
  - ClusterEnsembles
  - scTriangulate
  - scTriangulate 0.9.0
  - scanpy
  - scanpy 1.7.2
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# clustering-performance-metrics-evaluation

## Summary

Evaluates single-cell clustering solutions (e.g., scTriangulate integrated assignments vs. individual Leiden predictions) by computing Homogeneity, Completeness, and V-Measure (harmonic mean) against silver-standard reference labels, as done in the scTriangulate multiome PBMC benchmark.

## When to use

Use when a multi-clustering integration result (scTriangulate clusters) must be quantitatively compared against silver-standard references or individual modality/resolution clustering solutions using V-Measure, Homogeneity, and Completeness.

## When NOT to use

- You only have a single clustering solution and no reference labels to compare against — internal stability scores (reassign, TFIDF5, SCCAF) inside scTriangulate are the appropriate tools instead
- The task is cluster discovery or marker interpretation (e.g., identifying classical/intermediate/inflammatory CD14 monocytes) rather than quantitative agreement evaluation

## Inputs

- Integrated scTriangulate cluster assignment (final annotation column in AnnData.obs) for the multiome PBMC nuclei
- Individual Leiden clustering annotations (RNA resolutions 1/2/3, ATAC resolutions 1/2/3) as label vectors
- Silver-standard reference cell identity assignments (e.g., reference label transfer annotations)

## Outputs

- multiome_pbmc_sctriangulate_metrics.csv containing Homogeneity, Completeness, and V-Measure per evaluated clustering solution
- Comparison table supporting Supplementary Fig. 5d (scTriangulate vs. individual clustering predictions)

## How to apply

After running scTriangulate lazy_run (version 0.9.0) on the 10x Multiome PBMC data — with RNA and ATAC Leiden solutions at resolutions 1, 2, and 3 concatenated via concat_rna_and_other, win_fraction_cutoff=0.35 and add_metrics='tfidf5' — treat the integrated cluster assignment and each input annotation as label vectors over the same ~10,991 retained nuclei. Compute Homogeneity, Completeness, and their harmonic mean (V-Measure) for the scTriangulate solution and for each individual clustering prediction against silver-standard reference-assigned cell identities. Use the Python package ClusterEnsembles with default options for ensemble-style agreement computation, as in the paper's methods. Write the per-comparison metric table to multiome_pbmc_sctriangulate_metrics.csv and verify scTriangulate's V-Measure/Completeness reproduce the reported superiority over individual clustering predictions (Supplementary Fig. 5d).

## Related tools

- **ClusterEnsembles** (Python package used with default options for computing clustering agreement metrics between predictions) — https://github.com/tsano430/ClusterEnsembles
- **scTriangulate 0.9.0** (Produces the integrated cell-population assignments whose V-Measure/Completeness are benchmarked against individual clustering solutions) — https://github.com/frankligy/scTriangulate
- **scanpy 1.7.2** (Provides the Leiden clustering solutions and preprocessing pipeline whose outputs feed the metric evaluation)

## Examples

```
from sklearn.metrics import homogeneity_completeness_v_measure; h, c, v = homogeneity_completeness_v_measure(adata.obs['ref_labels'], adata.obs['sctriangulate'])
```

## Evaluation signals

- multiome_pbmc_sctriangulate_metrics.csv exists and contains Homogeneity, Completeness, and V-Measure fields for each evaluated clustering solution
- All label vectors cover the same ~10,991 nuclei retained after QC (RNA min_genes>=300, min_counts>=500, pct_counts_mt<=20%, >=1000 ATAC peaks per nucleus)
- scTriangulate's V-Measure and Completeness values reproduce the reported superiority over individual RNA/ATAC Leiden clustering predictions (Supplementary Fig. 5d)
- V-Measure equals the harmonic mean of the computed Homogeneity and Completeness (2*H*C/(H+C)); consistency check per row
- Reference-assignment and metric computation use the ClusterEnsembles package with default options, matching the paper's methods

## Limitations

- Programmatic default analytical settings may not be optimal for all study designs
- Agreement metrics require a silver-standard reference; conclusions depend on reference quality (population naming remains largely non-standardized, an unmet challenge)
- Integration of cluster predictions from dozens of sources significantly increases runtime before metrics can be computed

## Evidence

- [methods] ClusterEnsembles default options for metric computation: "using the Python package ClusterEnsembles with default options (https://github.com/tsano430/ClusterEnsembles)."
- [abstract] scTriangulate benchmark superiority claim: "When evaluated on diverse multimodal technologies, scTriangulate outperforms alternative approaches to identify high-confidence cell-populations and modality-specific subtypes."
- [methods] Software versions used for preprocessing and QC: "All the preprocessing and QC steps were conducted using scTriangulate version 0.9.0 and scanpy version 1.7.2."
- [readme] Reproducibility materials availability: [evidence span withheld — anchor status: external_doc]
