---
name: cross-cohort-component-correlation-analysis
description: Use when a component model (e.g., 86 MeCs learned by ICA from integrated
  TME scRNA-seq) must be shown to be reproducible rather than an artifact of any single
  training dataset, by testing exclusion of each cohort in turn.
when_to_use_negative:
- The learned components are being generated for the first time with no full-data
  reference set available to correlate against.
- The task is per-dataset integration/batch correction of cell representations (e.g.,
  Harmony/scVI/scArches joint embeddings) rather than testing component-set robustness
  under cohort exclusion.
- Input is a single cohort with no multi-dataset structure, since leave-one-out across
  cohorts is undefined.
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: MetaTiME
  role: Provides the metatime.mecmapper ICA-based MeC discovery module used to learn
    MeC sets on full data and on each leave-one-out subset, plus the reference 86
    MeC space to correlate against
  repo: github.com/yi-zhang/MetaTiME
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-023-38333-8
    title: MetaTiME integrates single-cell gene expression to characterize the meta-components
      of the tumor immune microenvironment
  source_content_hash: sha256:5545af733cc5c8ccd69f7152d09b248e26b2f2cb09007b80c1aaabd398ef7ff2
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MetaTiME
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# cross-cohort-component-correlation-analysis

## Summary

Assess whether data-learned transcriptional components (e.g., MetaTiME MeCs) are robust to cohort composition by re-learning components under leave-one-out cohort removal and correlating each perturbed component set against the full-data reference set. This quantifies reproducibility of latent gene-expression programs learned from integrated single-cell data.

## When to use

Use when a component model (e.g., 86 MeCs learned by ICA from integrated TME scRNA-seq) must be shown to be reproducible rather than an artifact of any single training dataset, by testing exclusion of each cohort in turn.

## When NOT to use

- The learned components are being generated for the first time with no full-data reference set available to correlate against.
- The task is per-dataset integration/batch correction of cell representations (e.g., Harmony/scVI/scArches joint embeddings) rather than testing component-set robustness under cohort exclusion.
- Input is a single cohort with no multi-dataset structure, since leave-one-out across cohorts is undefined.

## Inputs

- Curated integrated training expression matrix (2,157,387 TME cells aggregated from 93 datasets, TISCH-derived)
- Full-set reference MeC component set (86 meta-components learned via ICA on all data)
- Per-dataset cohort labels used to define leave-one-out splits

## Outputs

- Per-dataset leave-one-out MeC sets (ICA components re-learned without each cohort)
- Per-run maximum correlation of leave-one-out MeCs against the 86-MeC full-set reference
- Per-run MeC counts compared with the full-set count of 86
- Aggregated robustness metrics (mean max correlation 0.988; mean leave-one-out MeC number 84.64) and a table such as leave_one_out_robustness.csv

## How to apply

Load the integrated training expression matrix (here 2,157,387 tumor microenvironment cells aggregated across 93 TISCH-derived datasets) used by the metatime.mecmapper ICA-based MeC discovery module. For each of the 93 datasets in turn, drop that dataset and re-run ICA-based MeC learning on the remaining cells to produce a leave-one-out component set. Correlate every leave-one-out MeC against the full-set reference of 86 MeCs and record the maximum correlation per run; count MeCs recovered per run. Aggregate across runs — MetaTiME reports mean maximum correlation 0.988 and mean leave-one-out MeC number 84.64 versus 86 for the full set — and write per-dataset values to a table (e.g., leave_one_out_robustness.csv). High maximum correlations and near-equal component counts are the acceptance criteria demonstrating the MeCs reflect independent gene-expression directions rather than cohort idiosyncrasies.

## Related tools

- **MetaTiME** (Provides the metatime.mecmapper ICA-based MeC discovery module used to learn MeC sets on full data and on each leave-one-out subset, plus the reference 86 MeC space to correlate against) — github.com/yi-zhang/MetaTiME

## Examples

```
pip install metatime  # then use the metatime.mecmapper MeC discovery module to re-run ICA MeC learning per leave-one-out subset and correlate against the 86 full-set MeCs
```

## Evaluation signals

- Mean maximum correlation between each leave-one-out MeC set and the full-set reference should be near the reported 0.988 (i.e., close to 1).
- Mean number of leave-one-out MeCs should approximate the full-set count of 86 (reported 84.64); large per-run deviations flag cohort-specific components.
- Per-dataset table (e.g., leave_one_out_robustness.csv) should contain one maximum-correlation and one MeC-count row for each of the 93 excluded datasets.
- Every leave-one-out run must be executed on the identical curated matrix minus one dataset (2,157,387 total cells), ensuring split integrity.

## Limitations

- Robustness is evaluated against the full-set reference learned from the same underlying data distribution, so it does not guarantee generalization to entirely unseen cohorts or cancer types.
- Leave-one-out testing requires re-running ICA-based learning 93 times (once per dataset), which is computationally expensive on ~2.16-million-cell matrices.
- Maximum-correlation summary can mask individual MeCs with low reproducibility; per-component inspection of the correlation table is advisable.

## Evidence

- [intro] MetaTiME identifies reproducible low-dimensional meta-components that reflect independent components of gene expression variation across cohorts and cancer types.: [evidence span withheld — anchor status: not_found]
- [intro] These MeCs represent the TME landscape along 86 data-driven transcriptional directions mirroring lineage-specific cell states and signaling activities.: "These MeCs represent the TME landscape along 86 data-driven transcriptional directions mirroring lineage-specific cell states and signaling activities."
- [discussion] performed a large-scale and pan-cancer integration of tumor single cell datasets using ICA to optimize information independence among components51.: "performed a large-scale and pan-cancer integration of tumor single cell datasets using ICA to optimize information independence among components51."
- [readme] MetaTiME learns data-driven, interpretable, and reproducible gene programs by integrating millions of single cells from hundreds of tumor scRNA-seq data.: "MetaTiME learns data-driven, interpretable, and reproducible gene programs by integrating millions of single cells from hundreds of tumor scRNA-seq data."
