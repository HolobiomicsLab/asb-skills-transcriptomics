---
name: leave-one-out-resampling-robustness-analysis
description: Use when a learned representation (e.g., MetaTiME MeCs trained on multi-cohort
  single-cell data) must be shown to be robust to the removal of any single training
  cohort before it is reused for annotation.
when_to_use_negative:
- The goal is annotating new TME scRNA-seq data with pretrained MeCs via the MetaTiME
  Annotator — use the mapping/projection skill instead of re-training.
- Only one training cohort exists, so leave-one-out across cohorts is undefined and
  stability cannot be assessed by cohort exclusion.
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: MetaTiME
  role: Provides the ICA-based mecmapper MeC discovery module and pretrained MeC framework
    used to re-learn components under each leave-one-out split and to correlate them
    against the 86-component full-set reference.
  repo: github.com/yi-zhang/MetaTiME
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-023-38333-8
    title: MetaTiME integrates single-cell gene expression to characterize the meta-components
      of the tumor immune microenvironment
  source_content_hash: sha256:302691a6f1a52a99a69e5ab576d3e196685ed807bca6957388c86947cdd690b8
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MetaTiME
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# leave-one-out-resampling-robustness-analysis

## Summary

A resampling protocol that re-runs ICA-based MetaTiME meta-component (MeC) learning 93 times, each time excluding one of the 93 TISCH-derived training datasets, to test whether the 86 full-set MeCs are stable to the removal of any single cohort.

## When to use

Use when a learned representation (e.g., MetaTiME MeCs trained on multi-cohort single-cell data) must be shown to be robust to the removal of any single training cohort before it is reused for annotation.

## When NOT to use

- The goal is annotating new TME scRNA-seq data with pretrained MeCs via the MetaTiME Annotator — use the mapping/projection skill instead of re-training.
- Only one training cohort exists, so leave-one-out across cohorts is undefined and stability cannot be assessed by cohort exclusion.

## Inputs

- TISCH-derived training single-cell expression matrix (2,157,387 cells, 93 tumor scRNA-seq datasets) used by metatime.mecmapper
- Full-set reference MeC component set (86 meta-components) learned on the complete data
- Per-dataset cohort labels identifying which dataset to exclude in each run

## Outputs

- 93 leave-one-out MeC component sets
- Per-dataset table of maximum correlations against the full-set MeCs and recovered MeC counts
- leave_one_out_robustness.csv with aggregated metrics (mean max correlation 0.988; mean MeC number 84.64 vs 86)

## How to apply

Load the curated TISCH-derived training expression matrix (2,157,387 cells aggregated across 93 datasets) consumed by the metatime.mecmapper MeC discovery module. For each of the 93 datasets in turn, exclude that dataset and re-run the ICA-based MeC learning on the remaining cells, since the original components were derived by large-scale pan-cancer integration of tumor single-cell datasets using ICA to optimize information independence. Correlate each leave-one-out MeC set against the full-set reference of 86 interpretable MeCs and record the maximum correlation per run. Count the MeCs recovered per run and compare with the full-set count of 86. Aggregate across runs: the expected result is a mean maximum correlation of 0.988 and a mean leave-one-out MeC number of 84.64 vs 86 for the full set, supporting that MeCs reflect stable gene expression components across cohorts and cancer types.

## Related tools

- **MetaTiME** (Provides the ICA-based mecmapper MeC discovery module and pretrained MeC framework used to re-learn components under each leave-one-out split and to correlate them against the 86-component full-set reference.) — github.com/yi-zhang/MetaTiME

## Examples

```
pip install metatime  # then iterate over the 93 cohorts in the TISCH-derived matrix, re-running the metatime.mecmapper ICA MeC learning per exclusion
```

## Evaluation signals

- Mean maximum correlation of leave-one-out MeC sets against the full-set 86 MeCs reaches ~0.988.
- Mean number of recovered leave-one-out MeCs is ~84.64, close to the full-set count of 86.
- Every per-dataset run yields a maximum correlation against the full-set reference close to 1 (no single cohort removal catastrophically changes the component space).
- leave_one_out_robustness.csv contains one row per excluded dataset (93 rows) with max-correlation and MeC-count columns populated.

## Limitations

- Robustness is assessed by correlation against the full-set reference, not against independent biological validation; a component absent from a held-out cohort cannot be detected by matching.
- Each of the 93 runs requires re-learning components from ~2 million cells, so the protocol is computationally expensive and depends on the full TISCH-derived training matrix being available (distributed via Zenodo record 7410180).
- Results apply to the tumor immune microenvironment (TME) single-cell context in which MetaTiME was trained; stability numbers may not transfer to other tissues or omics.

## Evidence

- [readme] MetaTiME learns data-driven, interpretable, and reproducible gene programs by integrating millions of single cells from hundreds of tumor scRNA-seq data.: "MetaTiME learns data-driven, interpretable, and reproducible gene programs by integrating millions of single cells from hundreds of tumor scRNA-seq data."
- [intro] MetaTiME identifies reproducible low-dimensional meta-components that reflect independent components of gene expression variation across cohorts and cancer types.: [evidence span withheld — anchor status: not_found]
- [discussion] performed a large-scale and pan-cancer integration of tumor single cell datasets using ICA to optimize information independence among components: "performed a large-scale and pan-cancer integration of tumor single cell datasets using ICA to optimize information independence among components51."
- [discussion] We identified 86 interpretable meta-components (MeCs) that describe common aspects of TME gene expression variation across multiple tumors.: "We identified 86 interpretable meta-components (MeCs) that describe common aspects of TME gene expression variation across multiple tumors."
- [readme] Tumor scRNAseq Data for MetaTiME @ [Zenodo](https://zenodo.org/record/7410180): [evidence span withheld — anchor status: external_doc]
