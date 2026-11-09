---
name: immunotherapy-response-cohort-comparison
description: Use when comparing tumor microenvironment scRNA-seq cohorts split by
  immunotherapy condition (e.g., BCC pre vs post-ICB, BLCA ICB responders vs non-responders)
  to test which cell-state or signaling signature programs differ between groups.
when_to_use_negative:
- Input is bulk RNA-seq or a precomputed deconvolution reference (CIBERSORT-style)
  rather than single-cell TME expression to be projected onto the MeC space
- The data are non-tumor single-cell datasets outside the tumor immune microenvironment
  scope that the pretrained MeCs were trained on
- The analysis goal is de novo discovery of latent components rather than scoring
  and differential testing against the fixed pretrained 86-MeC space
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: MetaTiME
  role: Projects each cohort's expression matrix onto the pretrained MeC space (metatime.mecmapper,
    metatime.annotator) to score the 86 meta-components, then supports differential
    signature activity testing across immunotherapy response groups
  repo: https://github.com/yi-zhang/MetaTiME
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-023-38333-8
    title: MetaTiME integrates single-cell gene expression to characterize the meta-components
      of the tumor immune microenvironment
  source_content_hash: sha256:b3f9b8edabc78ddb191ad3ea6105252bbd019e41baa072e04be69912cfb8f32a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/yi-zhang/MetaTiME
  tools:
  - MetaTiME
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 15
---

# immunotherapy-response-cohort-comparison

## Summary

A statistical comparison skill that projects pre/post-ICB and responder/non-responder tumor scRNA-seq cohorts onto MetaTiME's pretrained 86 meta-component (MeC) space, then runs per-cluster two-sided t-tests on MeC signature scores to reveal differential cell-state programs across immunotherapy response groups.

## When to use

Use when comparing tumor microenvironment scRNA-seq cohorts split by immunotherapy condition (e.g., BCC pre vs post-ICB, BLCA ICB responders vs non-responders) to test which cell-state or signaling signature programs differ between groups.

## When NOT to use

- Input is bulk RNA-seq or a precomputed deconvolution reference (CIBERSORT-style) rather than single-cell TME expression to be projected onto the MeC space
- The data are non-tumor single-cell datasets outside the tumor immune microenvironment scope that the pretrained MeCs were trained on
- The analysis goal is de novo discovery of latent components rather than scoring and differential testing against the fixed pretrained 86-MeC space

## Inputs

- Tumor microenvironment scRNA-seq cohorts with immunotherapy condition labels (e.g., TISCH BCC pre/post-ICB, BLCA responder/non-responder)
- Pretrained MetaTiME meta-component (MeC) model covering 86 interpretable components
- Cell-state cluster assignments per cohort

## Outputs

- Per-cell MeC signature scores (86 components) for each cohort
- Per-cluster, per-condition aggregated MeC signature score table
- Significance–effect-size results for all MeCs from two-sided t-tests (e.g., mecs_differential_ttest_results.csv)

## How to apply

Load each cohort (e.g., BCC pre-/post-ICB and BLCA responder/non-responder scRNA-seq from TISCH) using the metatime.loaddata module, then project each cohort's expression matrix onto the pretrained MeC space with metatime.mecmapper and metatime.annotator to compute per-cell scores for the 86 meta-components. Aggregate the MeC signature scores by cell cluster and condition group (pre vs post ICB; responder vs non-responder). Run per-cluster two-sided t-tests on each MeC signature score between condition groups, compute effect sizes, and compile the significance–effect-size table (e.g., mecs_differential_ttest_results.csv). The rationale is that MetaTiME's data-driven MeCs encode independent, cross-cohort gene expression directions (cell types, cell states, signaling activities), so differences in MeC scores reflect reproducible transcriptional program shifts rather than study-specific marker choices.

## Related tools

- **MetaTiME** (Projects each cohort's expression matrix onto the pretrained MeC space (metatime.mecmapper, metatime.annotator) to score the 86 meta-components, then supports differential signature activity testing across immunotherapy response groups) — https://github.com/yi-zhang/MetaTiME

## Examples

```
pip install metatime
```

## Evaluation signals

- Projection yields per-cell scores for all 86 meta-components for each cohort
- Elevated cytotoxic and B-cell MeC signatures are detected post-ICB in the BCC cohort
- MeC-42 (M_Macrophage-IL1-NFKB, IL1B-positive macrophage signature) is elevated in BLCA ICB non-responders vs responders via differential MeC signature testing
- Two-sided t-tests are run per cell-state cluster and per MeC, with effect sizes compiled alongside p-values into mecs_differential_ttest_results.csv
- Aggregated MeC scores partition correctly by condition group (pre vs post ICB in BCC; responder vs non-responder in BLCA)

## Limitations

- MeCs are pretrained, so transcriptional programs outside the 86 learned directions will not be captured by this comparison
- The repository README notes it is continuously being improved and details may change; dependency versions are pinned (e.g., pandas==1.1.5, scanpy==1.8.2, anndata==0.8.0)
- Prior cell-type/state definitions used for comparison varied across studies; conclusions depend on the MeC annotation framework rather than manually curated marker sets

## Evidence

- [intro] we develop a MetaTiME toolkit for using the MeCs to annotate cellular states and signature continuums in tumor scRNA datasets, and to reveal differential signatures across immunotherapy responses.: "we develop a MetaTiME toolkit for using the MeCs to annotate cellular states and signature continuums in tumor scRNA datasets, and to reveal differential signatures across immunotherapy responses."
- [discussion] MetaTiME provides a toolkit for analyzing independent TME scRNA-seq datasets by mapping gene expression onto the MeC space.: "MetaTiME provides a toolkit for analyzing independent TME scRNA-seq datasets by mapping gene expression onto the MeC space."
- [readme] MetaTiME provides pretrained meta-components (MeCs) to automatically annotate fine-grained cell states and plot signature continuum for new single-cells: "MetaTiME provides pretrained meta-components (MeCs) to automatically annotate fine-grained cell states and plot signature continuum for new single-cells of tumor microenvironment."
- [intro] MetaTiME identifies reproducible low-dimensional meta-components that reflect independent components of gene expression variation across cohorts and cancer types.: [evidence span withheld — anchor status: not_found]
