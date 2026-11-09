---
name: ica-meta-component-decomposition-single-cell
description: Use when you have a large pan-cancer tumor scRNA-seq expression matrix
  (e.g. thousands to millions of cells across many cohorts) and want reproducible,
  low-dimensional gene programs for cell-state annotation rather than marker-based
  manual labelling.
when_to_use_negative:
- Input is already a projected cell-state annotation or feature-signature table rather
  than raw gene-by-cell scRNA-seq expression data — the ICA decomposition requires
  expression matrices.
- The goal is joint low-dimensional integration of a small set of query datasets via
  methods like Harmony or scArches; MetaTiME MeC learning instead trains a reusable
  component space from a large aggregated corpus and maps new data onto it.
- The data are not tumor microenvironment scRNA-seq; the MeC space is trained on TME
  single cells and its cell-type/state interpretation is TME-specific.
tools:
- name: MetaTiME
  role: Provides the ICA-based MeC discovery module (metatime.mecmapper) and pretrained
    meta-components used for the decomposition and robustness testing
  repo: github.com/yi-zhang/MetaTiME
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-023-38333-8
    title: MetaTiME integrates single-cell gene expression to characterize the meta-components
      of the tumor immune microenvironment
  source_content_hash: sha256:c672e87c1dcaa3451a1a527811af9db48c876cab38a36495401c819d7eac387c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MetaTiME
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# ica-meta-component-decomposition-single-cell

## Summary

Learns data-driven, interpretable meta-components (MeCs) of the tumor immune microenvironment by running ICA-based decomposition over millions of TME single-cell gene expression profiles, yielding ~86 transcriptional directions that encode independent gene-expression variation across cancer types.

## When to use

Use when you have a large pan-cancer tumor scRNA-seq expression matrix (e.g. thousands to millions of cells across many cohorts) and want reproducible, low-dimensional gene programs for cell-state annotation rather than marker-based manual labelling.

## When NOT to use

- Input is already a projected cell-state annotation or feature-signature table rather than raw gene-by-cell scRNA-seq expression data — the ICA decomposition requires expression matrices.
- The goal is joint low-dimensional integration of a small set of query datasets via methods like Harmony or scArches; MetaTiME MeC learning instead trains a reusable component space from a large aggregated corpus and maps new data onto it.
- The data are not tumor microenvironment scRNA-seq; the MeC space is trained on TME single cells and its cell-type/state interpretation is TME-specific.

## Inputs

- Uniformly processed tumor scRNA-seq gene-by-cell expression matrix (TISCH-derived; reference scale: 2,157,387 cells across 93 datasets)
- Dataset/cohort metadata specifying the 93 training datasets for leave-one-out splitting
- Reference immune cell type gene signatures (Azimuth, CIBERSORT, Human Primary Cell Atlas) for component validation

## Outputs

- Set of 86 ICA-derived meta-components (MeCs) as gene-loading vectors
- leave_one_out_robustness.csv: per-dataset table of maximum correlation to the full-set MeCs and leave-one-out MeC counts
- Correlation scores of MeCs against external immune cell type reference databases

## How to apply

Assemble a uniformly processed tumor single-cell training matrix (the MetaTiME reference run aggregates 2,157,387 cells from 93 TISCH-derived datasets) and run the ICA-based MeC learning in the metatime.mecmapper module; ICA is chosen to optimize information independence among components. The full-set decomposition yields 86 interpretable MeCs representing lineage-specific cell states and signaling activities of the TME. To validate robustness, repeat the decomposition 93 times under leave-one-out testing (exclude each training dataset in turn, re-run ICA on the remaining cells), correlate each leave-one-out component set against the full-set 86 MeCs, and record maximum correlation and component count per run. Aggregate the 93 runs into a robustness table (expected: mean maximum correlation 0.988; mean leave-one-out MeC number 84.64 vs 86 for the full set) written to leave_one_out_robustness.csv. The learned components can then be validated biologically by correlating MeCs with reference immune cell type databases such as Azimuth, CIBERSORT signatures, and the Human Primary Cell Atlas.

## Related tools

- **MetaTiME** (Provides the ICA-based MeC discovery module (metatime.mecmapper) and pretrained meta-components used for the decomposition and robustness testing) — github.com/yi-zhang/MetaTiME

## Examples

```
pip install metatime
```

## Evaluation signals

- Full-set ICA decomposition recovers 86 interpretable meta-components matching the reported reference component count
- Leave-one-out validation: mean maximum correlation of leave-one-out MeC sets against the full-set reference reaches 0.988
- Leave-one-out component counts stay close to the full set (mean 84.64 vs 86)
- Learned MeCs show meaningful correlations with established immune cell type references (Azimuth, CIBERSORT, Human Primary Cell Atlas)
- Decomposition input integrity check: training matrix contains the expected scale of cells (~2,157,387 across 93 datasets)

## Limitations

- Robustness depends on the large aggregated training corpus; results on much smaller or fewer-cohort inputs are not established by this evidence.
- MeC learning requires the full training matrix (2,157,387 cells, 93 datasets) — leave-one-out evaluation requires 93 re-runs of the ICA step, which is computationally expensive.

## Evidence

- [readme] MetaTiME learns data-driven, interpretable, and reproducible gene programs by integrating millions of single cells from hundreds of tumor scRNA-seq data.: "MetaTiME learns data-driven, interpretable, and reproducible gene programs by integrating millions of single cells from hundreds of tumor scRNA-seq data."
- [discussion] performed a large-scale and pan-cancer integration of tumor single cell datasets using ICA to optimize information independence among components: "performed a large-scale and pan-cancer integration of tumor single cell datasets using ICA to optimize information independence among components51."
- [intro] These MeCs represent the TME landscape along 86 data-driven transcriptional directions mirroring lineage-specific cell states and signaling activities.: "These MeCs represent the TME landscape along 86 data-driven transcriptional directions mirroring lineage-specific cell states and signaling activities."
