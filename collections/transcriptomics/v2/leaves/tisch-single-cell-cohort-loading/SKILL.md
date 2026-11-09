---
name: tisch-single-cell-cohort-loading
description: Use when an analysis requires downloading and loading a tumor microenvironment
  scRNA-seq cohort with immunotherapy condition annotations (pre/post ICB, responder/non-responder)
  as input for MetaTiME MeC scoring or differential signature testing.
when_to_use_negative:
- Input is already a per-cell MeC score table or feature matrix rather than raw/processed
  per-cell expression from a tumor cohort — go directly to MeC differential testing
  instead
- Data are non-tumor single-cell datasets or bulk expression; the pretrained MeC space
  and TISCH loading path target tumor microenvironment (TME) scRNA-seq
- A new joint latent representation across cohorts is required (e.g., integration
  with Harmony or scVI), rather than mapping onto the fixed pretrained 86-MeC space
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: MetaTiME
  role: Provides the metatime.loaddata module used to load TISCH scRNA-seq cohorts,
    and the downstream mecmapper/annotator modules that consume the loaded expression
    matrices; installable via pip from PyPI.
  repo: https://github.com/yi-zhang/MetaTiME
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-023-38333-8
    title: MetaTiME integrates single-cell gene expression to characterize the meta-components
      of the tumor immune microenvironment
  source_content_hash: sha256:b385a92f821ec697b36758692d2db56a03c537bc999ecaa74021cb49e5049065
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/yi-zhang/MetaTiME
  tools:
  - MetaTiME
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# tisch-single-cell-cohort-loading

## Summary

Load uniformly processed tumor scRNA-seq cohorts (e.g., BCC pre/post-ICB and BLCA ICB responder/non-responder) from the TISCH collection using the MetaTiME metatime.loaddata module, producing per-cell expression matrices ready for projection onto the pretrained MeC space.

## When to use

Use when an analysis requires downloading and loading a tumor microenvironment scRNA-seq cohort with immunotherapy condition annotations (pre/post ICB, responder/non-responder) as input for MetaTiME MeC scoring or differential signature testing.

## When NOT to use

- Input is already a per-cell MeC score table or feature matrix rather than raw/processed per-cell expression from a tumor cohort — go directly to MeC differential testing instead
- Data are non-tumor single-cell datasets or bulk expression; the pretrained MeC space and TISCH loading path target tumor microenvironment (TME) scRNA-seq
- A new joint latent representation across cohorts is required (e.g., integration with Harmony or scVI), rather than mapping onto the fixed pretrained 86-MeC space

## Inputs

- Cohort identifiers for TISCH tumor scRNA-seq datasets (e.g., BCC pre/post-ICB cohort, BLCA responder/non-responder cohort)
- Per-cell condition metadata (pre vs post ICB; ICB responder vs non-responder)
- Expression matrix in processed TISCH form compatible with scanpy/anndata

## Outputs

- Loaded per-cell scRNA-seq expression matrices for each cohort (scanpy/anndata objects)
- Cell-by-condition group labels (BCC pre/post, BLCA responder/non-responder) for downstream per-cluster t-tests
- Cohorts ready for projection onto the 86 MeC pretrained space via metatime.mecmapper/metatime.annotator

## How to apply

Install the metatime package (pip install metatime; tested with pandas==1.1.5, scanpy==1.8.2, anndata==0.8.0, matplotlib==3.5.1, adjustText==0.7.3, leidenalg==0.8.3) and use its loaddata module to fetch the target cohort (e.g., BCC pre-/post-ICB or BLCA responder/non-responder) from the TISCH uniformly processed tumor single-cell collection. Keep cohort-level condition metadata (pre vs post ICB; responder vs non-responder) attached to the per-cell expression matrix because downstream steps aggregate MeC signature scores by cluster and condition group. Once loaded, hand the expression matrix to metatime.mecmapper and metatime.annotator to compute per-cell scores for the 86 meta-components. Uniform preprocessing across cohorts matters because MetaTiME's pretrained MeC space was learned from millions of TME single cells across cancer types and assumes comparably processed input.

## Related tools

- **MetaTiME** (Provides the metatime.loaddata module used to load TISCH scRNA-seq cohorts, and the downstream mecmapper/annotator modules that consume the loaded expression matrices; installable via pip from PyPI.) — https://github.com/yi-zhang/MetaTiME

## Examples

```
pip install metatime; import metatime; adata = metatime.loaddata.load_cohort('BLCA')
```

## Evaluation signals

- The loaded cohort retains per-cell condition labels (pre vs post ICB for BCC; responder vs non-responder for BLCA) so MeC scores can be aggregated by cluster and condition group
- The expression matrix dimensions match the expected TISCH cohort cell/gene counts and can be projected onto the pretrained MeC space without errors from metatime.mecmapper
- Per-cell MeC scores for all 86 meta-components are produced for every loaded cell, enabling the subsequent per-cluster two-sided t-tests
- Dependency versions match the README-tested environment (pandas==1.1.5, scanpy==1.8.2, anndata==0.8.0) to avoid load-time incompatibilities
- Downstream verification reproduces the expected differential results: elevated cytotoxic and B-cell MeC signatures post-ICB in BCC and elevated MeC-42 (M_Macrophage-IL1-NFKB) in BLCA non-responders

## Limitations

- The repository README notes it is 'continously being improved' and that 'More details will be updated and suggested improvements welcome', so loaddata API details may shift across versions
- Dependency versions are only tested up to pandas==1.1.5, scanpy==1.8.2, anndata==0.8.0, matplotlib==3.5.1, adjustText==0.7.3, leidenalg==0.8.3; newer environments may break loading
- TISCH cohorts must be the uniformly processed tumor single-cell collection matching MetaTiME's training-data preprocessing; datasets processed differently may not map correctly onto the MeC space

## Evidence

- [other] Load the BCC pre-/post-ICB and BLCA responder/non-responder scRNA-seq cohorts from TISCH using the metatime.loaddata module.: [evidence span withheld — anchor status: not_found]
- [readme] MetaTiME provides pretrained meta-components for new TME scRNA-seq data.: "MetaTiME provides pretrained meta-components (MeCs) to automatically annotate fine-grained cell states and plot signature continuum for new single-cells of tumor microenvironment."
- [discussion] Map gene expression onto the MeC space to analyze independent TME datasets.: "MetaTiME provides a toolkit for analyzing independent TME scRNA-seq datasets by mapping gene expression onto the MeC space."
- [readme] Tumor scRNAseq training data available on Zenodo record 7410180.: [evidence span withheld — anchor status: external_doc]
