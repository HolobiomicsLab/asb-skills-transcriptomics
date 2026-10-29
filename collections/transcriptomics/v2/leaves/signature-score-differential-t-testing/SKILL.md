---
name: signature-score-differential-t-testing
description: Use when scRNA-seq data from tumor immune microenvironment samples has
  been scored for MetaTiME MeC signatures and the goal is to compare per-cell-state-cluster
  signature activity across two condition groups (pre/post treatment, responder/non-responder).
when_to_use_negative:
- Input scRNA-seq data is not tumor-microenvironment-derived or is non-human — the
  pretrained MeC space was learned from tumor TME single cells and may not transfer
- The goal is dataset integration or joint embedding rather than testing per-cluster
  differential signature activity between condition groups
- Only one condition group is available, leaving no pair to compare in the two-sided
  t-tests
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: MetaTiME
  role: Provides the pretrained 86-MeC space and the metatime.loaddata, metatime.mecmapper,
    and metatime.annotator modules used to score new TME scRNA-seq data so differential
    t-tests can be run on shared, interpretable signature continuums
  repo: https://github.com/yi-zhang/MetaTiME
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-023-38333-8
    title: MetaTiME integrates single-cell gene expression to characterize the meta-components
      of the tumor immune microenvironment
  source_content_hash: sha256:67bc4621f1ec68b0ff5b11ad7b15978cfc97f9ede237bc2dc67f3019d6f9aea7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/yi-zhang/MetaTiME
  tools:
  - MetaTiME
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# signature-score-differential-t-testing

## Summary

Per-cluster two-sided t-testing of MetaTiME meta-component (MeC) signature scores between condition groups (e.g., ICB responders vs non-responders) to detect differential transcriptional programs in tumor immune microenvironment scRNA-seq data. Grounded in MetaTiME's pretrained 86-MeC space, which transfers cell-state knowledge to new TME datasets.

## When to use

Use when scRNA-seq data from tumor immune microenvironment samples has been scored for MetaTiME MeC signatures and the goal is to compare per-cell-state-cluster signature activity across two condition groups (pre/post treatment, responder/non-responder).

## When NOT to use

- Input scRNA-seq data is not tumor-microenvironment-derived or is non-human — the pretrained MeC space was learned from tumor TME single cells and may not transfer
- The goal is dataset integration or joint embedding rather than testing per-cluster differential signature activity between condition groups
- Only one condition group is available, leaving no pair to compare in the two-sided t-tests

## Inputs

- Tumor microenvironment scRNA-seq cohorts with two condition groups (e.g., BCC pre/post-ICB, BLCA responder/non-responder from TISCH)
- Expression matrix (scanpy/anndata object) per cohort
- Pretrained MetaTiME 86 meta-component (MeC) projection model
- Cell cluster / cell-state annotations per cohort

## Outputs

- Per-cell MeC signature scores for 86 meta-components
- Per-cluster two-sided t-test p-values comparing each MeC signature score between condition groups
- Effect sizes per MeC per cluster
- Significance–effect-size results table (mecs_differential_ttest_results.csv)

## How to apply

Load the TME scRNA-seq cohorts (e.g., BCC pre/post-ICB, BLCA responder/non-responder from TISCH) using metatime.loaddata, then project each cohort's expression matrix onto the pretrained MeC space with metatime.mecmapper and metatime.annotator to compute per-cell scores for all 86 meta-components. Aggregate MeC signature scores by cell cluster and condition group (e.g., pre vs post ICB in BCC; responder vs non-responder in BLCA). Run per-cluster two-sided t-tests on each MeC signature score between the condition groups, then compute effect sizes and compile significance and effect-size results for all MeCs into a results table (mecs_differential_ttest_results.csv). The rationale is that MeCs encode independent, pan-cancer gene-expression directions, so per-cluster t-testing in this shared space reveals interpretable differential cell-state signatures (e.g., cytotoxic and B-cell MeCs elevated post-ICB in BCC; MeC-42, M_Macrophage-IL1-NFKB, elevated in BLCA non-responders) without re-learning new gene programs from the test data.

## Related tools

- **MetaTiME** (Provides the pretrained 86-MeC space and the metatime.loaddata, metatime.mecmapper, and metatime.annotator modules used to score new TME scRNA-seq data so differential t-tests can be run on shared, interpretable signature continuums) — https://github.com/yi-zhang/MetaTiME

## Examples

```
pip install metatime; import metatime.loaddata, metatime.mecmapper, metatime.annotator; scores = mecmapper.project(expr_matrix); annotator.score_signature_continuum(scores)
```

## Evaluation signals

- Reproduces the expected differential result: cytotoxic and B-cell MeC signatures elevated post-ICB in the BCC pre/post-ICB cohort
- Reproduces the expected differential result: MeC-42 (M_Macrophage-IL1-NFKB, IL1B-positive macrophage signature) elevated in BLCA ICB non-responders versus responders
- All 86 MeC signature scores appear in the output table with both t-test significance and effect-size columns populated
- Tests are performed per cell-state cluster and are two-sided (both directions of differential signature activity reported)
- Per-cell scores fall within the expected range of the pretrained MetaTiME MeC space when projecting the expression matrix via metatime.mecmapper

## Limitations

- Differential MeC signatures can only be interpreted within the 86 components learned by MetaTiME; cell-state programs outside this space will not be captured
- MetaTiME annotation and signature scoring depend on pretrained components tuned to tumor immune microenvironment data (dependencies tested with pandas==1.1.5, scanpy==1.8.2, anndata==0.8.0); other dataset versions may break the pipeline
- Two-sided t-tests per cluster do not control for multiple testing across 86 MeCs and clusters beyond what the workflow explicitly implements

## Evidence

- [abstract] By projecting onto the MetaTiME space, we provide a tool to annotate cell states and signature continuums for TME scRNA-seq data.: "By projecting onto the MetaTiME space, we provide a tool to annotate cell states and signature continuums for TME scRNA-seq data."
- [intro] Use MetaTiMe MeCs to annotate cellular states and reveal differential signatures across immunotherapy responses: "we develop a MetaTiME toolkit for using the MeCs to annotate cellular states and signature continuums in tumor scRNA datasets, and to reveal differential signatures across immunotherapy responses."
- [readme] README: test differential signature activity tutorial: [evidence span withheld — anchor status: external_doc]
- [discussion] 86 interpretable meta-components identified: "We identified 86 interpretable meta-components (MeCs) that describe common aspects of TME gene expression variation across multiple tumors."
