---
name: gene-zweight-averaging-into-mec-profiles
description: Use when a cohort-shared ICA gene z-weight matrix (task_inputs port 'ica_zweight_matrix')
  is available and the goal is to collapse reproducible ICs into consensus meta-components
  for tumor microenvironment annotation.
when_to_use_negative:
- Input is raw single-cell expression data rather than a cohort-shared ICA gene z-weight
  matrix — upstream ICA and reproducibility filtering must be run first (see MetaTiMEpretrain
  pipeline).
- The goal is annotating new TME scRNA-seq data with pretrained MeCs; use the MetaTiME-Annotator
  mapping instead of re-deriving the MeC space.
- Components are expected to be cohort-specific (e.g. a single-dataset factor analysis)
  where cross-cohort reproducibility filtering (≥3 datasets Pearson correlation) would
  incorrectly discard them.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: MetaTiME
  role: Framework whose discovery stage filters reproducible ICs and builds consensus
    MeCs; provides pretrained MeCs and the retraining pipeline
  repo: https://github.com/yi-zhang/MetaTiME
- name: leidenalg
  role: Community detection dependency used for clustering related ICs (Louvain/Leiden-style
    graph clustering) in the MetaTiME package
  repo: https://github.com/yi-zhang/MetaTiME
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-023-38333-8
    title: MetaTiME integrates single-cell gene expression to characterize the meta-components
      of the tumor immune microenvironment
  source_content_hash: sha256:1809d02b0d76cf055f31a79a88290be202d0e303037e8132d0cfa3cead779b36
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/yi-zhang/MetaTiME
  tools:
  - MetaTiME
  - leidenalg
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# gene-zweight-averaging-into-mec-profiles

## Summary

Derives MetaTiME meta-components (MeCs) by retaining ICA independent components whose gene z-weight profiles are reproducible across cohorts (minimum Pearson correlation matched in ≥3 datasets), clustering them via Louvain community detection (resolution 1.25, clusters <5 discarded), and averaging within-cluster gene z-weights into MeC profiles.

## When to use

Use when a cohort-shared ICA gene z-weight matrix (task_inputs port 'ica_zweight_matrix') is available and the goal is to collapse reproducible ICs into consensus meta-components for tumor microenvironment annotation.

## When NOT to use

- Input is raw single-cell expression data rather than a cohort-shared ICA gene z-weight matrix — upstream ICA and reproducibility filtering must be run first (see MetaTiMEpretrain pipeline).
- The goal is annotating new TME scRNA-seq data with pretrained MeCs; use the MetaTiME-Annotator mapping instead of re-deriving the MeC space.
- Components are expected to be cohort-specific (e.g. a single-dataset factor analysis) where cross-cohort reproducibility filtering (≥3 datasets Pearson correlation) would incorrectly discard them.

## Inputs

- cohort-shared ICA gene z-weight matrix (task_inputs port 'ica_zweight_matrix'): independent components (ICs) as rows/columns with gene z-weight features

## Outputs

- mec_profiles.csv: MeC gene-weight profile table with one row per MeC and gene z-weight columns
- on the order of 86 interpretable meta-components (MeCs) describing common aspects of TME gene expression variation

## How to apply

Load the cohort-shared ICA gene z-weight matrix and compute pairwise cosine-distance similarity between the gene z-weight vectors of independent components (ICs) across cohorts to define component-to-component similarity. Retain only ICs whose z-weight profiles are reproducible, requiring a minimum Pearson correlation with matching components in at least 3 datasets, so downstream MeCs reflect shared rather than cohort-specific programs. Build a similarity graph over the retained ICs and run Louvain community detection at resolution 1.25, discarding clusters smaller than 5 components. Average the gene z-weights of all ICs within each Louvain cluster to define each meta-component (MeC) profile, then write the MeC gene-weight profile table to mec_profiles.csv. The rationale is MetaTiME's premise that learned meta-components encode independent gene expression programs observed across cancer types, so consensus over correlated ICs yields interpretable, transferable directions rather than cohort-specific noise.

## Related tools

- **MetaTiME** (Framework whose discovery stage filters reproducible ICs and builds consensus MeCs; provides pretrained MeCs and the retraining pipeline) — https://github.com/yi-zhang/MetaTiME
- **leidenalg** (Community detection dependency used for clustering related ICs (Louvain/Leiden-style graph clustering) in the MetaTiME package) — https://github.com/yi-zhang/MetaTiME

## Examples

```
pip install metatime
```

## Evaluation signals

- The number of derived MeCs reproduces the scale of the published meta-component space (86 interpretable meta-components reported for MetaTiME).
- Every output row of mec_profiles.csv contains a defined MeC profile over the gene z-weight features (no empty/missing profiles).
- All ICs contributing to the graph passed the reproducibility filter: minimum Pearson correlation with matching components in at least 3 datasets.
- No Louvain cluster smaller than 5 components was retained (resolution 1.25, min cluster size 5).
- Each retained MeC profile equals the average gene z-weight of the ICs in its Louvain cluster (checkable by re-averaging cluster memberships).

## Limitations

- Reproducibility filtering discards ICs not correlated across at least 3 cohorts, so cohort-specific expression programs are intentionally excluded from the MeC space.
- Clusters smaller than 5 ICs are discarded at resolution 1.25, so rare programs may not become MeCs.
- Supporting annotations and some pipeline details live in external artifacts (e.g. MetaTiMEpretrain repo, Supplementary Data) not contained in the main text.
- GR ChIP-seq support for relevant T cell states is unavailable in some contexts; binding is inferred from other cell lines — interpret regulator-linked MeCs cautiously.

## Evidence

- [intro] MetaTiME identifies reproducible low-dimensional meta-components that reflect independent components of gene expression variation across cohorts and cancer types.: [evidence span withheld — anchor status: not_found]
- [discussion] 86 interpretable meta-components were identified: "We identified 86 interpretable meta-components (MeCs) that describe common aspects of TME gene expression variation across multiple tumors."
- [discussion] ICA integration across cohorts: "performed a large-scale and pan-cancer integration of tumor single cell datasets using ICA to optimize information independence among components51."
- [readme] README training data description: "MetaTiME learns data-driven, interpretable, and reproducible gene programs by integrating millions of single cells from hundreds of tumor scRNA-seq data."
