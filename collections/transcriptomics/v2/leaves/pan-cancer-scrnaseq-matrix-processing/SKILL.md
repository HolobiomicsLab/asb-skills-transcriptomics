---
name: pan-cancer-scrnaseq-matrix-processing
description: Use when a large collection of uniformly processed tumor scRNA-seq matrices
  (e.g., TISCH-derived, 2,157,387 cells across 93 datasets) must be integrated into
  interpretable gene-expression components that transfer across cohorts and cancer
  types.
when_to_use_negative:
- The goal is joint embedding of existing cohorts for batch-corrected visualization
  rather than learning transferable gene-expression components — integration algorithms
  such as Harmony or scVI/scArches are the article's contrast cases
- The input is a small single-study scRNA-seq matrix; the MeC space is learned from
  millions of cells across hundreds of tumor datasets and should be applied, not re-derived,
  from such data
- Cell-type annotation from known gene markers suffices — the article positions MetaTiME
  as overcoming limits of manual marker-based labelling, so marker-based workflows
  are an alternative rather than this skill
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: MetaTiME
  role: Framework providing the ICA-based MeC discovery module (metatime.mecmapper)
    and the toolkit for projecting scRNA-seq data onto the learned MeC space for annotation
  repo: github.com/yi-zhang/MetaTiME
- name: MetaTiMEpretrain
  role: Re-training pipeline for re-running MeC learning (e.g., leave-one-out runs)
    on the aggregated training matrix
  repo: https://github.com/yi-zhang/MetaTiMEpretrain/
- name: scanpy
  role: Dependency for handling scRNA-seq matrices (AnnData) in the MetaTiME environment
- name: anndata
  role: Dependency providing the single-cell expression matrix container used by MetaTiME
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-023-38333-8
    title: MetaTiME integrates single-cell gene expression to characterize the meta-components
      of the tumor immune microenvironment
  source_content_hash: sha256:226f0b676db9975108db16804c6e0312c5b831e2f33d0625c05b6a6ebcd29a06
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MetaTiME
  - MetaTiMEpretrain
  - anndata
  - scanpy
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# pan-cancer-scrnaseq-matrix-processing

## Summary

Integrates a large-scale, pan-cancer collection of tumor single-cell RNA-seq matrices using ICA to learn reproducible low-dimensional meta-components (MeCs) of the tumor immune microenvironment. This is the core matrix-processing step behind MetaTiME's 86 data-driven transcriptional directions.

## When to use

Use when a large collection of uniformly processed tumor scRNA-seq matrices (e.g., TISCH-derived, 2,157,387 cells across 93 datasets) must be integrated into interpretable gene-expression components that transfer across cohorts and cancer types.

## When NOT to use

- The goal is joint embedding of existing cohorts for batch-corrected visualization rather than learning transferable gene-expression components — integration algorithms such as Harmony or scVI/scArches are the article's contrast cases
- The input is a small single-study scRNA-seq matrix; the MeC space is learned from millions of cells across hundreds of tumor datasets and should be applied, not re-derived, from such data
- Cell-type annotation from known gene markers suffices — the article positions MetaTiME as overcoming limits of manual marker-based labelling, so marker-based workflows are an alternative rather than this skill

## Inputs

- Curated TISCH-derived tumor scRNA-seq training expression matrix (2,157,387 cells across 93 datasets)
- Dataset membership labels mapping each cell to its source dataset (for leave-one-out splits)
- Full-set reference MeC component matrix (86 MeCs) for correlation-based robustness checks

## Outputs

- 86 data-driven meta-components (MeCs) as gene loading vectors
- leave_one_out_robustness.csv with per-dataset maximum correlations and leave-one-out MeC counts
- Pretrained MeC space for projecting new TME scRNA-seq data (MetaTiME scores for TME cells, distributed via Zenodo record 7410180)

## How to apply

Load the curated TISCH-derived training expression matrix (2,157,387 cells aggregated across 93 tumor scRNA-seq datasets) consumed by the metatime.mecmapper MeC discovery module, using the preprocessed data distributed via Zenodo (record 7410180). Run ICA-based decomposition across the pan-cancer data to optimize information independence among components, yielding 86 data-driven transcriptional directions that mirror lineage-specific cell states and signaling activities. To assess robustness, perform leave-one-out re-training: exclude each of the 93 datasets in turn, re-run the ICA-based MeC learning on the remaining cells, and correlate each leave-one-out component set against the full-set reference of 86 MeCs, recording the maximum correlation per run. Expect a mean maximum correlation of 0.988 and a mean leave-one-out MeC number of 84.64 versus 86 for the full set; write per-dataset maximum correlations and MeC counts to leave_one_out_robustness.csv. For applying learned components to new data, use the MetaTiME toolkit to project expression onto the MeC space for cell-state annotation and signature continuums.

## Related tools

- **MetaTiME** (Framework providing the ICA-based MeC discovery module (metatime.mecmapper) and the toolkit for projecting scRNA-seq data onto the learned MeC space for annotation) — github.com/yi-zhang/MetaTiME
- **MetaTiMEpretrain** (Re-training pipeline for re-running MeC learning (e.g., leave-one-out runs) on the aggregated training matrix) — https://github.com/yi-zhang/MetaTiMEpretrain/
- **scanpy** (Dependency for handling scRNA-seq matrices (AnnData) in the MetaTiME environment)
- **anndata** (Dependency providing the single-cell expression matrix container used by MetaTiME)

## Examples

```
pip install metatime  # then re-train MeCs per leave-one-out split with the MetaTiMEpretrain pipeline: https://github.com/yi-zhang/MetaTiMEpretrain/
```

## Evaluation signals

- Full-set ICA run recovers 86 interpretable meta-components, matching the published MeC count
- Across all 93 leave-one-out runs, mean maximum correlation against the full-set MeCs is ~0.988
- Mean leave-one-out MeC number is ~84.64, close to the full-set count of 86 (each run should recover 80+ components)
- leave_one_out_robustness.csv contains one row per excluded dataset (93 rows) with a maximum-correlation column and a MeC-count column
- Learned MeCs correlate with reference immune cell type databases (e.g., Azimuth) and are interpretable as cell types, cell states, and signaling activities

## Limitations

- Robustness requires re-running ICA 93 times (one per excluded dataset); full reproduction depends on access to the Zenodo training data (record 7410180)
- Training data is tumor/TME scRNA-seq; the learned MeC space is oriented to the tumor immune microenvironment and may not transfer to non-TME tissues
- MetaTiME MeCs encode pan-cancer common variation; components specific to a single held-out cohort may drop out (mean 84.64 vs 86 components in leave-one-out testing)
- Tested dependency versions are specific (pandas==1.1.5, scanpy==1.8.2, anndata==0.8.0, matplotlib==3.5.1, adjustText==0.7.3, leidenalg==0.8.3); other versions may not behave identically

## Evidence

- [intro] MetaTiME identifies reproducible low-dimensional meta-components that reflect independent components of gene expression variation across cohorts and cancer types.: [evidence span withheld — anchor status: not_found]
- [intro] These MeCs represent the TME landscape along 86 data-driven transcriptional directions mirroring lineage-specific cell states and signaling activities.: "These MeCs represent the TME landscape along 86 data-driven transcriptional directions mirroring lineage-specific cell states and signaling activities."
- [discussion] performed a large-scale and pan-cancer integration of tumor single cell datasets using ICA to optimize information independence among components51.: "performed a large-scale and pan-cancer integration of tumor single cell datasets using ICA to optimize information independence among components51."
- [readme] MetaTiME learns data-driven, interpretable, and reproducible gene programs by integrating millions of single cells from hundreds of tumor scRNA-seq data.: "MetaTiME learns data-driven, interpretable, and reproducible gene programs by integrating millions of single cells from hundreds of tumor scRNA-seq data."
- [readme] Tumor scRNAseq Data for MetaTiME @ Zenodo - A large collection of uniformly processed tumor single-cell RNA-seq. Includes raw data and MetaTiME score for the TME cells.: "Tumor scRNAseq Data for MetaTiME @ [Zenodo](https://zenodo.org/record/7410180)

- A large collection of uniformly processed tumor single-cell RNA-seq."
