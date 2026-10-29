---
name: cms-subtype-fraction-tabulation
description: Use when CMS class assignments from CMScaller (or classifyCMS.RF) must
  be summarized as per-dataset subtype fractions across pre-clinical CRC datasets
  to verify subtype representation.
when_to_use_negative:
- Input is single-cell RNA-sequencing data — CMScaller is not optimized for it because
  scRNA-seq is exceedingly noisy
- Samples have a different human stromal component than primary CRCs, e.g. metastatic
  CRC biopsies, where CMScaller is not recommended
- Datasets with fewer than approximately forty samples, where the required centering
  and scaling inescapably introduces prediction uncertainty
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_2640
- http://edamontology.org/topic_0203
tools:
- name: CMScaller
  role: Cancer cell-intrinsic classifier used to assign CMS1–CMS4 classes to all 551
    pre-clinical samples so subtype fractions can be tabulated
  repo: https://github.com/Lothelab/CMScaller
- name: R
  role: Environment in which CMScaller classification, classifyCMS.RF comparison,
    and fraction tabulation are performed
- name: Biobase
  role: Used to load centered and scaled expression matrices as ExpressionSet objects
    required as CMScaller input
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41598-017-16747-x
    title: 'CMScaller: an R package for consensus molecular subtyping of colorectal
      cancer pre-clinical models'
  source_content_hash: sha256:26cf72af5c740b06c174503c9f3a200ac8d6dfd8827536dff449740e32bbcd08
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Lothelab/CMScaller
  tools:
  - Biobase
  - CMScaller
  - R
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_2640
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# cms-subtype-fraction-tabulation

## Summary

Tabulate CMS1–CMS4 class fractions per colorectal cancer pre-clinical dataset (cell lines, organoids, PDXs) after CMScaller classification, and quantify the original classifyCMS.RF failure mode (unclassified/CMS2-dominant calls) as a comparator.

## When to use

Use when CMS class assignments from CMScaller (or classifyCMS.RF) must be summarized as per-dataset subtype fractions across pre-clinical CRC datasets to verify subtype representation.

## When NOT to use

- Input is single-cell RNA-sequencing data — CMScaller is not optimized for it because scRNA-seq is exceedingly noisy
- Samples have a different human stromal component than primary CRCs, e.g. metastatic CRC biopsies, where CMScaller is not recommended
- Datasets with fewer than approximately forty samples, where the required centering and scaling inescapably introduces prediction uncertainty

## Inputs

- CMScaller class-prediction results (per-sample CMS1–CMS4 calls) for 551 in vitro and in vivo models
- classifyCMS.RF prediction outputs with default settings on the same 551 samples
- Per-dataset sample lists mapping the seven public datasets (GSE59857, GSE76402, GSE35144, GSE64392, GSE74843, E-MTAB-991, Gao PDX) to samples

## Outputs

- Per-dataset table of CMS1–CMS4 sample fractions (counts and percentages)
- Comparator tally of classifyCMS.RF outcomes: unclassified, CMS2, and CMS4 counts out of 551 samples
- Accept/reject verdict against the 7.5–22% CMS4 fraction and 219/245/3 count criteria

## How to apply

After running CMScaller on each of the seven public pre-clinical expression datasets (GSE59857, GSE76402, GSE35144, GSE64392, GSE74843, E-MTAB-991, Gao PDX) — with each expression matrix centered and scaled within dataset and loaded as an ExpressionSet via Biobase — cross-tabulate the predicted CMS1–CMS4 classes per dataset. Confirm that all four CMS groups are present in every dataset and that CMS4 accounts for 7.5–22% of samples. As a comparator, apply classifyCMS.RF with default settings to the same 551 samples and record unclassified (219), CMS2 (245), and CMS4 (3) call counts to reproduce the stroma-dependent classifier's failure to detect CMS4 in cell lines, organoids and PDXs. The run is accepted when reproduced classifyCMS.RF counts match 219/245/3 and per-dataset CMS4 fractions fall within the 7.5–22% range.

## Related tools

- **CMScaller** (Cancer cell-intrinsic classifier used to assign CMS1–CMS4 classes to all 551 pre-clinical samples so subtype fractions can be tabulated) — https://github.com/Lothelab/CMScaller
- **R** (Environment in which CMScaller classification, classifyCMS.RF comparison, and fraction tabulation are performed)
- **Biobase** (Used to load centered and scaled expression matrices as ExpressionSet objects required as CMScaller input)

## Examples

```
res <- CMScaller(emat, RNAseq=TRUE); table(res$prediction, dataset=sampleMetadata$dataset); prop.table(table(res$prediction, sampleMetadata$dataset), 2)
```

## Evaluation signals

- Reproduced classifyCMS.RF counts equal 219 unclassified, 245 CMS2, and 3 CMS4 out of 551 samples
- All four CMS groups (CMS1–CMS4) are present in every one of the seven pre-clinical datasets
- Per-dataset CMS4 fractions fall within the reported 7.5–22% range
- CMScaller assigns a CMS class to all 551 samples (no unclassified calls, unlike the original classifier)

## Limitations

- The original classifier depends on immune and stromal gene expression signals and fails to identify the CMS4-mesenchymal group in immortalized cell lines, organoids and xenografts
- Highly selected datasets where molecular distributions deviate from primary CRC cohorts (including metastatic samples) warrant caution
- Small datasets (<~40 samples) inescapably introduce prediction uncertainty because inputs must be centered and scaled
- Intra-tumor heterogeneity (immune/stromal infiltration and cancer-cell intrinsic) may explain prediction errors at class boundaries

## Evidence

- [results] CMScaller classified all 551 pre-clinical samples such that all CMS groups were present in every dataset, with CMS4 accounting for 7.5–22% of samples: [evidence span withheld — anchor status: not_found]
- [abstract] The currently available classifier is dependent on gene expression signals from the immune and stromal compartments: "The currently available classifier is dependent on gene expression signals from the immune and stromal compartments of tumors and fails to identify the poor-prognostic CMS4-mesenchymal group"
- [discussion] input gene expression data should be centered and scaled. The implication is that small datasets inescapably introduce prediction uncertainty: "input gene expression data should be centered and scaled. The implication is that small datasets inescapably introduce prediction uncertainty"
- [abstract] This new classifier, referred to as CMScaller, recapitulated the subtypes in both in vitro and in vivo models (551 in total).: "This new classifier, referred to as CMScaller, recapitulated the subtypes in both in vitro and in vivo models (551 in total)."
- [readme] res <- CMScaller(crcTCGAsubset, RNAseq=TRUE, doPlot=TRUE): "res <- CMScaller(crcTCGAsubset, RNAseq=TRUE, doPlot=TRUE)"
