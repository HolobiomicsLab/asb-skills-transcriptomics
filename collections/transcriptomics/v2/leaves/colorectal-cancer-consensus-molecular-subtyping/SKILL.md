---
name: colorectal-cancer-consensus-molecular-subtyping
description: Use when gene expression matrices from colorectal cancer models (in vitro/in
  vivo) or TCGA pCRCs need CMS1-CMS4 labels, especially when immune/stromal signals
  are absent and the original CMS classifier fails to recover CMS1/CMS4.
when_to_use_negative:
- Samples with a different human stromal component than primary CRCs, e.g. patient
  biopsies of metastatic CRC, where CMScaller in its current implementation is not
  recommended
- Datasets with fewer than approximately forty samples, where centering/scaling inescapably
  introduces prediction uncertainty
- Single-cell RNA-sequencing data, which is exceedingly noisy and not what CMScaller
  is optimized for
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: CMScaller
  role: Core NTP-based classifier assigning CMS1-CMS4 labels from scaled expression
    matrices; also provides CMSgsa (Camera gene set analysis), subDEG and subVolcano
    for downstream evaluation
  repo: https://github.com/Lothelab/CMScaller
- name: R
  role: Runtime environment; CMScaller is an R package installed via devtools::install_github
- name: Biobase
  role: Bioconductor infrastructure used to load expression data as ExpressionSet
    objects for CMScaller input
provenance:
  source_task_ids:
  - task_002
  - task_008
  source_papers:
  - doi: 10.1038/s41598-017-16747-x
    title: 'CMScaller: an R package for consensus molecular subtyping of colorectal
      cancer pre-clinical models'
  source_content_hash: sha256:7064be88639a7d3f823baf4365154b3f174caf40838607ce76c682e09538f80c
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
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# colorectal-cancer-consensus-molecular-subtyping

## Summary

Classifies colorectal cancer pre-clinical models (cell lines, organoids, PDXs) and primary tumors into Consensus Molecular Subtypes (CMS1-CMS4) using CMScaller, a microenvironment-independent nearest-template-prediction (NTP) classifier built on cancer cell-intrinsic subtype-enriched markers, plus Camera-based gene set analysis (CMSgsa) for biological validation.

## When to use

Use when gene expression matrices from colorectal cancer models (in vitro/in vivo) or TCGA pCRCs need CMS1-CMS4 labels, especially when immune/stromal signals are absent and the original CMS classifier fails to recover CMS1/CMS4.

## When NOT to use

- Samples with a different human stromal component than primary CRCs, e.g. patient biopsies of metastatic CRC, where CMScaller in its current implementation is not recommended
- Datasets with fewer than approximately forty samples, where centering/scaling inescapably introduces prediction uncertainty
- Single-cell RNA-sequencing data, which is exceedingly noisy and not what CMScaller is optimized for

## Inputs

- Gene expression matrix from colorectal cancer pre-clinical models (cell lines, organoids, PDXs) or TCGA primary CRC samples, centered and scaled
- Expression matrices importable as ExpressionSet objects via Biobase
- Original CMS reference labels (e.g. for TCGA pCRC test set, n=143) when validating prediction accuracy

## Outputs

- Per-sample CMS1-CMS4 predictions with NTP confidence scores from CMScaller
- Camera gene set analysis results per subtype (e.g. cam$CMS4) from CMSgsa using CMS-informative gene sets
- Accuracy metrics: overall accuracy with 95% CI and class-wise sensitivity/specificity (e.g. written to cms_ntp_validation_metrics.csv)
- Plots: subtype prediction plots (doPlot=TRUE), gene set enrichment heatmaps (cmsgsa_heatmaps.pdf), subDEG volcano plots via subVolcano

## How to apply

Install CMScaller from GitHub (devtools::install_github('Lothelab/CMScaller')) with Biobase and limma dependencies. As input, provide a gene expression matrix (ExpressionSet-style; RNA-seq supported via RNAseq=TRUE) and center and scale the data, as CMScaller requires — noting this introduces prediction uncertainty for small datasets (fewer than ~40 samples). Run nearest template prediction with CMScaller using the cancer cell-intrinsic template gene sets (templates.CMS) to assign each sample a CMS1-CMS4 label with NTP confidence, optionally plotting results (doPlot=TRUE). To evaluate, compare NTP-assigned labels sample-by-sample against original CMS labels (e.g. TCGA pCRC test set, n=143) and compute overall accuracy with 95% CI and class-wise sensitivity/specificity — the reference result is 0.83 overall accuracy (95% CI 0.74-0.9). For biological validation, run CMSgsa (Camera rotation test via limma) on CMScaller-classified matrices and confirm the expected enrichment pattern: CMS1 MSI-like, CMS2 HNF4A/MYC/WNT activation, CMS3 CDX2/metabolic, CMS4 TGF-beta/EMT.

## Related tools

- **CMScaller** (Core NTP-based classifier assigning CMS1-CMS4 labels from scaled expression matrices; also provides CMSgsa (Camera gene set analysis), subDEG and subVolcano for downstream evaluation) — https://github.com/Lothelab/CMScaller
- **R** (Runtime environment; CMScaller is an R package installed via devtools::install_github)
- **Biobase** (Bioconductor infrastructure used to load expression data as ExpressionSet objects for CMScaller input)

## Examples

```
library(Biobase); library(CMScaller); res <- CMScaller(crcTCGAsubset, RNAseq=TRUE, doPlot=TRUE); cam <- CMSgsa(emat=crcTCGAsubset, class=res$prediction, RNAseq=TRUE)
```

## Evaluation signals

- On the TCGA pCRC held-out test set (n=143) with original CMS labels as reference, NTP-assigned labels should yield overall accuracy ~0.83 (95% CI 0.74-0.9), with reported class-wise metrics such as CMS3 sensitivity 0.71 and CMS4 specificity 0.91
- CMSgsa enrichment pattern matches expected subtype signatures: CMS1 MSI-like, CMS2 HNF4A/MYC/WNT activation, CMS3 CDX2/metabolic, CMS4 TGF-beta/EMT; validation checks cmsgsa_heatmaps.pdf exists and displays this pattern
- CMScaller output contains a 'prediction' column per sample (used as class input to CMSgsa and subDEG), with head(res) returning plausible CMS labels
- In pre-clinical models (551 in vitro/in vivo models in the paper), classified samples should recapitulate all four CMS groups including CMS1-immune and CMS4-mesenchymal, which the original classifier fails to identify
- cmsgsa output is a named list per CMS class (e.g. cam$CMS4) of per-gene-set camera statistics, non-empty for each subtype

## Limitations

- Not recommended for samples with a different human stromal component than primary CRCs, including patient biopsies of metastatic CRC
- Input must be centered and scaled, which introduces prediction uncertainty for small datasets (fewer than ~40 samples)
- Caution warranted in highly selected datasets where molecular distributions deviate from pCRC cohorts (e.g. metastatic samples)
- Not optimized for single-cell RNA-seq data, which is exceedingly noisy
- Intra-tumor heterogeneity (immune/stromal infiltration and cancer-cell intrinsic) may explain prediction errors at class boundaries

## Evidence

- [abstract] This new classifier, referred to as CMScaller, recapitulated the subtypes in both in vitro and in vivo models (551 in total).: "This new classifier, referred to as CMScaller, recapitulated the subtypes in both in vitro and in vivo models (551 in total)."
- [readme] README quick start shows the core calls: "res <- CMScaller(crcTCGAsubset, RNAseq=TRUE, doPlot=TRUE)"
- [readme] README describes CMSgsa Camera gene set analysis: "cam <- CMSgsa(emat=crcTCGAsubset, class=res$prediction, RNAseq=TRUE)"
- [discussion] Discussion caveat on stromal component and centering/scaling: "CMScaller in its current implementation is not recommended for use with samples with a different human stromal component than primary CRCs, including e.g. patient biopsies of metastatic CRC."
- [discussion] Discussion caveat on small datasets: "input gene expression data should be centered and scaled. The implication is that small datasets inescapably introduce prediction uncertainty"
