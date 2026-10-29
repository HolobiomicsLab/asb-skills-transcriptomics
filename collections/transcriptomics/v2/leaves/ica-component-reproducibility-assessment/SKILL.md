---
name: ica-component-reproducibility-assessment
description: Use when you have gene z-weight vectors for independent components derived
  from multiple tumor scRNA-seq cohorts and must keep only cross-cohort-reproducible
  components before building consensus meta-components.
when_to_use_negative:
- The input is single-cell data from only one cohort — no cross-cohort IC matching
  is possible, so the ≥3-dataset Pearson correlation filter is undefined.
- The goal is annotating new TME scRNA-seq data with the pretrained MeC space (use
  the MetaTiME-Annotator) rather than re-deriving the meta-components themselves.
- The input is already a consensus MeC gene-weight profile table (e.g. mec_profiles.csv);
  this skill produces that table, not one that consumes it.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: MetaTiME
  role: Framework that performs large-scale pan-cancer ICA integration of tumor single-cell
    datasets and derives the reproducible, interpretable meta-component space via
    this filtering and clustering step.
  repo: https://github.com/yi-zhang/MetaTiME
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-023-38333-8
    title: MetaTiME integrates single-cell gene expression to characterize the meta-components
      of the tumor immune microenvironment
  source_content_hash: sha256:a65569efb13fdeb8f93fbd6aa2d6bdd1fc23a2d81011576ecc752cd6f7b296ae
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/yi-zhang/MetaTiME
  tools:
  - MetaTiME
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# ica-component-reproducibility-assessment

## Summary

Filters independent components (ICs) learned by ICA across multiple tumor single-cell cohorts so that only transcriptional programs reproducible across datasets (minimum Pearson correlation with a matching component in ≥3 datasets) are retained, then merges them into consensus meta-components (MeCs) via Louvain clustering of the component similarity graph. This is how MetaTiME ensures its MeC space reflects shared, not cohort-specific, gene-expression variation.

## When to use

Use when you have gene z-weight vectors for independent components derived from multiple tumor scRNA-seq cohorts and must keep only cross-cohort-reproducible components before building consensus meta-components.

## When NOT to use

- The input is single-cell data from only one cohort — no cross-cohort IC matching is possible, so the ≥3-dataset Pearson correlation filter is undefined.
- The goal is annotating new TME scRNA-seq data with the pretrained MeC space (use the MetaTiME-Annotator) rather than re-deriving the meta-components themselves.
- The input is already a consensus MeC gene-weight profile table (e.g. mec_profiles.csv); this skill produces that table, not one that consumes it.

## Inputs

- Cohort-shared ICA gene z-weight matrix (independent component gene z-weight vectors across multiple tumor scRNA-seq cohorts; task_inputs port 'ica_zweight_matrix')

## Outputs

- mec_profiles.csv — MeC gene-weight profile table with one row per meta-component and gene z-weight columns
- Retained set of reproducible ICs (minimum Pearson correlation with matching components in ≥3 datasets)
- Louvain cluster assignments of retained ICs (resolution 1.25, clusters <5 components discarded)

## How to apply

Load the cohort-shared ICA gene z-weight matrix (task_inputs port 'ica_zweight_matrix') and compute pairwise cosine distances between the gene z-weight vectors of ICs across cohorts to define component-to-component similarity. Retain only ICs whose z-weight profiles are reproducible across cohorts, requiring a minimum Pearson correlation with matching components in at least 3 datasets — this removes cohort-specific variation so downstream MeCs are built from shared programs. Build a similarity graph over the retained ICs and run Louvain community detection at resolution 1.25, discarding clusters smaller than 5 components. Average the gene z-weights of all ICs within each Louvain cluster to define each meta-component (MeC) profile, and write the per-MeC gene z-weight table to mec_profiles.csv. The rationale is that ICA was run for pan-cancer integration to optimize information independence among components, so reproducibility filtering plus clustering collapses redundant, replicable ICs into interpretable MeCs.

## Related tools

- **MetaTiME** (Framework that performs large-scale pan-cancer ICA integration of tumor single-cell datasets and derives the reproducible, interpretable meta-component space via this filtering and clustering step.) — https://github.com/yi-zhang/MetaTiME

## Evaluation signals

- The number of derived MeCs reproduces the scale of the published meta-component space (86 interpretable meta-components reported for MetaTiME).
- Every retained IC has a minimum Pearson correlation with a matching component in at least 3 datasets.
- Each output row of mec_profiles.csv contains a defined MeC profile over the gene z-weight features (one row per MeC, gene z-weight columns).
- Louvain clustering was run at resolution 1.25 and no cluster smaller than 5 components was retained.
- Each MeC profile equals the average gene z-weight of all ICs within its Louvain cluster (check by recomputing cluster means).

## Limitations

- Cohort-specific transcriptional programs are deliberately discarded by the ≥3-dataset reproducibility requirement; components present in fewer cohorts cannot be recovered downstream.
- Re-training the MeC space is a separate pipeline (MetaTiME re-training pipeline at github.com/yi-zhang/MetaTiMEpretrain/), not part of the pip-installable annotation toolkit.
- The upstream ICA z-weight matrix must already be cohort-shared; the README lists tested dependency versions (e.g. pandas==1.1.5, scanpy==1.8.2, leidenalg==0.8.3) that may affect reproducibility of clustering.

## Evidence

- [intro] MetaTiME identifies reproducible low-dimensional meta-components that reflect independent components of gene expression variation across cohorts and cancer types.: [evidence span withheld — anchor status: not_found]
- [discussion] performed a large-scale and pan-cancer integration of tumor single cell datasets using ICA to optimize information independence among components51.: "performed a large-scale and pan-cancer integration of tumor single cell datasets using ICA to optimize information independence among components51."
- [discussion] We identified 86 interpretable meta-components (MeCs) that describe common aspects of TME gene expression variation across multiple tumors.: "We identified 86 interpretable meta-components (MeCs) that describe common aspects of TME gene expression variation across multiple tumors."
- [readme] MetaTiME learns data-driven, interpretable, and reproducible gene programs by integrating millions of single cells from hundreds of tumor scRNA-seq data.: "MetaTiME learns data-driven, interpretable, and reproducible gene programs by integrating millions of single cells from hundreds of tumor scRNA-seq data."
- [readme] [New] MetaTiME re-training pipeline at https://github.com/yi-zhang/MetaTiMEpretrain/: [evidence span withheld — anchor status: external_doc]
