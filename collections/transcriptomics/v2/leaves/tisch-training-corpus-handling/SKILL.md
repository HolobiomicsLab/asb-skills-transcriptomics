---
name: tisch-training-corpus-handling
description: Use when retraining or robustness-testing MetaTiME meta-components over
  the TISCH tumor scRNA-seq corpus — e.g., leave-one-out cohort exclusion, cross-cohort
  MeC correlation, or comparing recovered component counts against the 86-MeC full-set
  reference.
when_to_use_negative:
- You only need to annotate a new tumor scRNA-seq dataset with the pretrained MeCs
  — use the MetaTiME-Annotator projection workflow instead of touching the training
  corpus.
- Your data is not uniformly processed tumor microenvironment scRNA-seq; non-TME single-cell
  data was not part of the TISCH-derived training corpus and MeC stability was not
  characterized for it.
- You need a joint latent embedding across samples rather than independent components
  — integration methods like Harmony or scVI/scArches address that problem, while
  MetaTiME deliberately uses ICA for component independence.
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: MetaTiME
  role: Provides the metatime.mecmapper ICA-based MeC discovery module that consumes
    the TISCH training matrix and the pretrained MeC reference; retraining pipeline
    also maintained at the MetaTiMEpretrain repo
  repo: github.com/yi-zhang/MetaTiME
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-023-38333-8
    title: MetaTiME integrates single-cell gene expression to characterize the meta-components
      of the tumor immune microenvironment
  source_content_hash: sha256:c2819d2724b06777fb3b59babfd31b22be61393e59966b7b32314b1341986ff7
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MetaTiME
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# tisch-training-corpus-handling

## Summary

Loads and manipulates the curated TISCH-derived tumor scRNA-seq training corpus (2,157,387 cells aggregated across 93 datasets) consumed by MetaTiME's ICA-based MeC discovery module (metatime.mecmapper), enabling cohort-level operations such as leave-one-out retraining and component-stability comparison against the full-set reference of 86 meta-components.

## When to use

Use when retraining or robustness-testing MetaTiME meta-components over the TISCH tumor scRNA-seq corpus — e.g., leave-one-out cohort exclusion, cross-cohort MeC correlation, or comparing recovered component counts against the 86-MeC full-set reference.

## When NOT to use

- You only need to annotate a new tumor scRNA-seq dataset with the pretrained MeCs — use the MetaTiME-Annotator projection workflow instead of touching the training corpus.
- Your data is not uniformly processed tumor microenvironment scRNA-seq; non-TME single-cell data was not part of the TISCH-derived training corpus and MeC stability was not characterized for it.
- You need a joint latent embedding across samples rather than independent components — integration methods like Harmony or scVI/scArches address that problem, while MetaTiME deliberately uses ICA for component independence.

## Inputs

- Curated TISCH-derived training expression matrix (2,157,387 cells, uniformly processed tumor single-cell RNA-seq)
- Per-dataset cohort labels for the 93 tumor scRNA-seq datasets used for leave-one-out exclusion
- Full-set reference MeC set (86 meta-components learned on the complete data)

## Outputs

- Leave-one-out MeC component sets (one per excluded dataset)
- Per-dataset maximum correlation against the full-set 86-MeC reference
- Per-dataset leave-one-out MeC counts
- leave_one_out_robustness.csv aggregating maximum correlations and MeC counts across all 93 runs

## How to apply

Load the uniformly processed TISCH-derived training expression matrix (2,157,387 cells spanning 93 tumor datasets, distributed via Zenodo record 7410180) and track per-dataset membership so any cohort can be excluded. For each of the 93 datasets in turn, drop that cohort and re-run ICA-based MeC learning through metatime.mecmapper on the remaining cells, yielding a leave-one-out component set. Correlate each leave-one-out MeC set against the full-set reference of 86 MeCs, recording the maximum correlation per run, and count recovered components per run. The decision criterion is stability: the expected mean maximum correlation is 0.988 and the mean leave-one-out MeC number is 84.64 versus 86 for the full set, confirming that components reflect pan-cancer, cohort-independent structure rather than single-dataset idiosyncrasies.

## Related tools

- **MetaTiME** (Provides the metatime.mecmapper ICA-based MeC discovery module that consumes the TISCH training matrix and the pretrained MeC reference; retraining pipeline also maintained at the MetaTiMEpretrain repo) — github.com/yi-zhang/MetaTiME

## Examples

```
pip install metatime
```

## Evaluation signals

- Mean maximum leave-one-out correlation against the full-set 86-MeC reference should be ~0.988.
- Mean number of MeCs recovered across the 93 leave-one-out runs should be ~84.64, close to the full-set count of 86.
- Each of the 93 runs must exclude exactly one dataset and be trained only on the remaining cells of the 2,157,387-cell corpus.
- Output table leave_one_out_robustness.csv must contain one row per excluded dataset with maximum correlation and recovered MeC count.
- Recovered components should remain biologically interpretable, consistent with the 86 interpretable MeCs describing TME gene expression variation.

## Limitations

- Robustness figures (mean max correlation 0.988; mean 84.64 MeCs) were established specifically on the 93-dataset TISCH-derived corpus and need not transfer to other corpora or cell types.
- The training data are a specific uniformly processed Zenodo release (record 7410180); swapping in differently processed cohorts may change component stability.
- The repo is continuously being improved — dependency versions matter (tested stack includes pandas==1.1.5, scanpy==1.8.2, anndata==0.8.0, matplotlib==3.5.1, adjustText==0.7.3, leidenalg==0.8.3).

## Evidence

- [abstract] Using millions of TME single cells, MetaTiME learns meta-components that encode independent components of gene expression observed across cancer types.: "Using millions of TME single cells, MetaTiME learns meta-components that encode independent components of gene expression observed across cancer types."
- [intro] MetaTiME identifies reproducible low-dimensional meta-components that reflect independent components of gene expression variation across cohorts and cancer types.: [evidence span withheld — anchor status: not_found]
- [discussion] We identified 86 interpretable meta-components (MeCs) that describe common aspects of TME gene expression variation across multiple tumors.: "We identified 86 interpretable meta-components (MeCs) that describe common aspects of TME gene expression variation across multiple tumors."
- [discussion] performed a large-scale and pan-cancer integration of tumor single cell datasets using ICA to optimize information independence among components51.: "performed a large-scale and pan-cancer integration of tumor single cell datasets using ICA to optimize information independence among components51."
- [readme] MetaTiME learns data-driven, interpretable, and reproducible gene programs by integrating millions of single cells from hundreds of tumor scRNA-seq data.: "MetaTiME learns data-driven, interpretable, and reproducible gene programs by integrating millions of single cells from hundreds of tumor scRNA-seq data."
- [readme] Tumor scRNAseq Data for MetaTiME @ [Zenodo](https://zenodo.org/record/7410180): [evidence span withheld — anchor status: external_doc]
