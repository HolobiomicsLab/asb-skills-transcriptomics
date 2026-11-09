---
name: cross-cohort-single-cell-integration
description: Use when a pan-cancer single-cell integration stage has produced per-cohort
  ICA z-weight matrices and you must derive cohort-shared transcriptional meta-components
  instead of dataset-specific components.
when_to_use_negative:
- The goal is annotating cells in a new TME scRNA-seq dataset with pretrained components
  — use the MetaTiME-Annotator projection workflow instead of re-deriving MeCs.
- The analysis needs a joint low-dimensional embedding of cells across cohorts (e.g.
  Harmony/scVI-style integration) rather than interpretable gene-program components.
- Input is a single cohort's scRNA data with no multi-cohort ICA z-weight matrix,
  so reproducibility across ≥3 datasets cannot be assessed.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3295
tools:
- name: MetaTiME
  role: Framework whose discovery stage performs reproducible-IC filtering and Louvain
    clustering to build consensus MeCs; retraining pipeline available at MetaTiMEpretrain
  repo: https://github.com/yi-zhang/MetaTiME
- name: leidenalg
  role: Graph community-detection dependency used for clustering components in the
    similarity graph
  repo: https://github.com/yi-zhang/MetaTiME
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-023-38333-8
    title: MetaTiME integrates single-cell gene expression to characterize the meta-components
      of the tumor immune microenvironment
  source_content_hash: sha256:bebfc8525f4079e63a9c1769b0b382f2de3d3eba0f10a70c56ac8ac31e0fc66b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/yi-zhang/MetaTiME
  tools:
  - MetaTiME
  - leidenalg
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3295
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# cross-cohort-single-cell-integration

## Summary

Reproducibility-filter independent components (ICs) learned by ICA across multiple tumor scRNA-seq cohorts, cluster them into consensus meta-components (MeCs) via Louvain community detection, and average gene z-weights per cluster to define a shared, interpretable MeC space rather than cohort-specific programs.

## When to use

Use when a pan-cancer single-cell integration stage has produced per-cohort ICA z-weight matrices and you must derive cohort-shared transcriptional meta-components instead of dataset-specific components.

## When NOT to use

- The goal is annotating cells in a new TME scRNA-seq dataset with pretrained components — use the MetaTiME-Annotator projection workflow instead of re-deriving MeCs.
- The analysis needs a joint low-dimensional embedding of cells across cohorts (e.g. Harmony/scVI-style integration) rather than interpretable gene-program components.
- Input is a single cohort's scRNA data with no multi-cohort ICA z-weight matrix, so reproducibility across ≥3 datasets cannot be assessed.

## Inputs

- cohort-shared ICA gene z-weight matrix (one gene z-weight vector per independent component, from the upstream reproducibility-filter step; task_inputs port 'ica_zweight_matrix')

## Outputs

- mec_profiles.csv — MeC gene-weight profile table with one row per MeC and gene z-weight columns
- reproducibility-filtered IC set (ICs passing minimum Pearson correlation in ≥3 datasets)
- Louvain cluster assignments over the IC similarity graph (resolution 1.25, clusters of size ≥5)

## How to apply

Load the cohort-shared ICA gene z-weight matrix (task_inputs port 'ica_zweight_matrix'). Compute pairwise cosine-distance similarity between the gene z-weight vectors of independent components (ICs) across cohorts to define component-to-component similarity, and retain only ICs reproducible across cohorts, requiring a minimum Pearson correlation with matching components in at least 3 datasets. Build a similarity graph over the retained ICs and run Louvain community detection at resolution 1.25, discarding clusters smaller than 5 components. Average the gene z-weights of all ICs within each Louvain cluster to define each meta-component (MeC) profile; the rationale is that reproducibility filtering plus graph clustering ensures MeCs encode independent gene-expression components shared across cancer types rather than cohort-specific variation. Write the resulting gene-weight profile table to mec_profiles.csv, one row per MeC with gene z-weight columns, targeting the scale of the published 86 interpretable meta-components.

## Related tools

- **MetaTiME** (Framework whose discovery stage performs reproducible-IC filtering and Louvain clustering to build consensus MeCs; retraining pipeline available at MetaTiMEpretrain) — https://github.com/yi-zhang/MetaTiME
- **leidenalg** (Graph community-detection dependency used for clustering components in the similarity graph) — https://github.com/yi-zhang/MetaTiME

## Examples

```
pip install metatime
```

## Evaluation signals

- Number of derived MeCs reproduces the scale of the published meta-component space (86 interpretable meta-components reported for MetaTiME).
- Every retained IC has a minimum Pearson correlation with matching components in at least 3 datasets.
- No Louvain cluster smaller than 5 components contributes an MeC (resolution 1.25 clustering).
- mec_profiles.csv schema check: one row per MeC, each with a defined gene z-weight profile over the z-weight feature columns.

## Limitations

- The reproducibility threshold (correlation in ≥3 datasets) depends on having enough cohorts; fewer than 3 comparable datasets cannot support shared MeC discovery.
- Discarding clusters smaller than 5 components at resolution 1.25 may drop rare or cohort-restricted programs by design.
- IC reproducibility is assessed on ICA z-weight profiles; components optimized for information independence may still require the graph-clustering merge to avoid redundancy.

## Evidence

- [intro] MetaTiME identifies reproducible low-dimensional meta-components that reflect independent components of gene expression variation across cohorts and cancer types.: [evidence span withheld — anchor status: not_found]
- [discussion] performed a large-scale and pan-cancer integration of tumor single cell datasets using ICA to optimize information independence among components51.: "performed a large-scale and pan-cancer integration of tumor single cell datasets using ICA to optimize information independence among components51."
- [intro] These MeCs represent the TME landscape along 86 data-driven transcriptional directions mirroring lineage-specific cell states and signaling activities.: "These MeCs represent the TME landscape along 86 data-driven transcriptional directions mirroring lineage-specific cell states and signaling activities."
- [readme] MetaTiME learns data-driven, interpretable, and reproducible gene programs by integrating millions of single cells from hundreds of tumor scRNA-seq data.: "MetaTiME learns data-driven, interpretable, and reproducible gene programs by integrating millions of single cells from hundreds of tumor scRNA-seq data."
