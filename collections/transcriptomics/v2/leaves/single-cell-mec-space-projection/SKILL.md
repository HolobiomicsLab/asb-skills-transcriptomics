---
name: single-cell-mec-space-projection
description: Use when you have a tumor scRNA-seq dataset (e.g., TISCH cohorts like
  BCC pre/post-ICB or BLCA responder/non-responder) and need consistent, interpretable
  cell-state scores rather than manual marker-based labeling.
when_to_use_negative:
- The data is not tumor microenvironment single-cell expression — the pretrained MeC
  space was learned from tumor scRNA-seq and may not transfer to non-TME contexts.
- The goal is de novo discovery of gene programs or latent components — MetaTiME projects
  onto pretrained, fixed MeCs rather than re-learning components (use the MetaTiMEpretrain
  pipeline instead if retraining is intended).
- The input is already a projected score table or a summary feature matrix rather
  than per-cell expression data.
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: MetaTiME
  role: Provides the pretrained MeC space and the loaddata/mecmapper/annotator Python
    modules used to load cohorts, project expression, and annotate cell states
  repo: https://github.com/yi-zhang/MetaTiME
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-023-38333-8
    title: MetaTiME integrates single-cell gene expression to characterize the meta-components
      of the tumor immune microenvironment
  source_content_hash: sha256:5d2fb2147114c7a6a021d4e70b6f65e30fdec245b12ee9ca2c624f17a7b5282f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/yi-zhang/MetaTiME
  tools:
  - MetaTiME
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# single-cell-mec-space-projection

## Summary

Project tumor microenvironment scRNA-seq expression matrices onto MetaTiME's pretrained 86 meta-component (MeC) space to compute per-cell scores that can be annotated as cell states and tested for differential signature activity. This transfers a data-driven, pan-cancer latent representation learned from millions of TME single cells to new datasets.

## When to use

Use when you have a tumor scRNA-seq dataset (e.g., TISCH cohorts like BCC pre/post-ICB or BLCA responder/non-responder) and need consistent, interpretable cell-state scores rather than manual marker-based labeling.

## When NOT to use

- The data is not tumor microenvironment single-cell expression — the pretrained MeC space was learned from tumor scRNA-seq and may not transfer to non-TME contexts.
- The goal is de novo discovery of gene programs or latent components — MetaTiME projects onto pretrained, fixed MeCs rather than re-learning components (use the MetaTiMEpretrain pipeline instead if retraining is intended).
- The input is already a projected score table or a summary feature matrix rather than per-cell expression data.

## Inputs

- tumor microenvironment scRNA-seq expression matrix (e.g., from TISCH, as an AnnData/count matrix)
- cohort condition labels (e.g., BCC pre-ICB vs post-ICB; BLCA responder vs non-responder)
- cell cluster assignments per dataset

## Outputs

- per-cell MeC signature scores for all 86 meta-components
- annotated cell-state labels per cell (from metatime.annotator)
- per-cluster differential MeC testing results (e.g., mecs_differential_ttest_results.csv with significance and effect sizes)

## How to apply

Install the metatime package via pip (tested with pandas==1.1.5, scanpy==1.8.2, anndata==0.8.0, matplotlib==3.5.1, adjustText==0.7.3, leidenalg==0.8.3, harmonypy). Load the cohort's expression matrix using the metatime.loaddata module, then project the expression matrix onto the pretrained MeC space with metatime.mecmapper and assign interpretable cell-state labels with metatime.annotator, yielding per-cell scores for all 86 meta-components (e.g., MeC-42 / M_Macrophage-IL1-NFKB). The MeCs are data-driven transcriptional directions mirroring lineage-specific cell states and signaling activities, so scores are directly comparable across cohorts and cancer types. Downstream, aggregate scores by cell cluster and condition group (pre vs post ICB; responder vs non-responder) and run per-cluster two-sided t-tests to detect differential signature activity. Success is verified when the projected space reproduces expected biology, e.g., elevated cytotoxic and B-cell MeC signatures post-ICB in BCC and elevated MeC-42 in BLCA non-responders.

## Related tools

- **MetaTiME** (Provides the pretrained MeC space and the loaddata/mecmapper/annotator Python modules used to load cohorts, project expression, and annotate cell states) — https://github.com/yi-zhang/MetaTiME

## Examples

```
pip install metatime
```

## Evaluation signals

- The projection returns scores for all 86 interpretable meta-components per cell
- Annotator output assigns biologically coherent cell-state labels (cell types, cell states, signaling activities) consistent with lineage markers
- Per-cluster two-sided t-tests on MeC scores reproduce expected differential results: elevated cytotoxic and B-cell MeC signatures post-ICB in BCC, and elevated MeC-42 (M_Macrophage-IL1-NFKB) in BLCA non-responders vs responders
- Significance and effect sizes for all MeCs are compiled into a complete results table (mecs_differential_ttest_results.csv) with no missing components
- Installation dependency versions match the tested set (e.g., scanpy==1.8.2, anndata==0.8.0)

## Limitations

- Cell type definitions from prior marker-based studies were chosen differently across studies, motivating MeC projection, but MeC scores are only meaningful within the TME scope of the pretrained space.
- The projection depends on the pretrained MeC model from Zenodo (record 7410180); datasets differing radically from the training distribution of hundreds of tumor scRNA-seq samples may project poorly.
- The repo is 'continuously being improved' and documentation is updated incrementally, so API details (e.g., module signatures) may change.

## Evidence

- [abstract] By projecting onto the MetaTiME space, we provide a tool to annotate cell states and signature continuums for TME scRNA-seq data.: "By projecting onto the MetaTiME space, we provide a tool to annotate cell states and signature continuums for TME scRNA-seq data."
- [intro] 86 data-driven transcriptional directions mirroring lineage-specific cell states: "These MeCs represent the TME landscape along 86 data-driven transcriptional directions mirroring lineage-specific cell states and signaling activities."
- [readme] MetaTiME provides pretrained meta-components (MeCs) to automatically annotate fine-grained cell states and plot signature continuum for new single-cells of tumor microenvironment.: "MetaTiME provides pretrained meta-components (MeCs) to automatically annotate fine-grained cell states and plot signature continuum for new single-cells of tumor microenvironment."
- [other] Project each cohort's expression matrix onto the pretrained MeC space with the metatime.mecmapper and metatime.annotator modules to compute per-cell scores for the 86 meta-components.: [evidence span withheld — anchor status: not_found]
- [discussion] MetaTiME provides a toolkit for analyzing independent TME scRNA-seq datasets by mapping gene expression onto the MeC space.: "MetaTiME provides a toolkit for analyzing independent TME scRNA-seq datasets by mapping gene expression onto the MeC space."
