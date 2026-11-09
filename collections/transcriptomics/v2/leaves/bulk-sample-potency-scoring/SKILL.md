---
name: bulk-sample-potency-scoring
description: Use when you have bulk RNA-Seq expression matrices and/or matched single-cell
  RNA-Seq profiles and need to score differentiation potency, identify cancer stem-cell
  or drug-resistant populations (e.
when_to_use_negative:
- When prior biological knowledge such as marker expression or timepoint is already
  reliable — SCENT is intended for scenarios where dropouts preclude marker use or
  hierarchies are not established
- When only a bulk-average profile (single homogeneous population) is analysed with
  no matched single-cell reference — the bulk-average negative control is expected
  to be non-significant (P≈0.30)
- When the goal is trajectory/branch inference rather than potency scoring — the article
  notes this insight is not readily obtainable from Monocle or MPath and conversely
  pseudo-time is not SCENT's output
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: SCENT
  role: Computes network-based signalling entropy (SR) of single cells and bulk samples
    and estimates differentiation potency
  repo: https://github.com/aet21/SCENT
- name: CCAT
  role: Used alongside SCENT in the tutorial for estimating differentiation potency
    of single cells
- name: R
  role: Runtime environment in which SCENT is installed and the paired Wilcoxon, Fisher,
    and permutation tests are run
- name: BiocStyle
  role: Bioconductor styling package used for the SCENT/CCAT tutorial vignette
provenance:
  source_task_ids:
  - task_006
  - task_007
  source_papers:
  - doi: 10.1038/ncomms15599
    title: Single-cell entropy for accurate estimation of differentiation potency
      from a cell’s transcriptome
  source_content_hash: sha256:e4d8e924c089ea13f611952cbc6f57d8da32cadcda7348338311fbe8932b502c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/aet21/SCENT
  tools:
  - BiocStyle
  - CCAT
  - R
  - SCENT
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# bulk-sample-potency-scoring

## Summary

Estimate the differentiation potency/plasticity of bulk RNA-Seq samples and single-cell populations by computing network-based signalling entropy (SR) with SCENT, then compare bulk SR against single-cell SR distributions to detect regulated expression heterogeneity (MRH) or drug-resistant stem-cell-like states.

## When to use

Use when you have bulk RNA-Seq expression matrices and/or matched single-cell RNA-Seq profiles and need to score differentiation potency, identify cancer stem-cell or drug-resistant populations (e.g. relapsed AML, enzalutamide-resistant CTCs), or test whether bulk entropy exceeds single-cell entropy.

## When NOT to use

- When prior biological knowledge such as marker expression or timepoint is already reliable — SCENT is intended for scenarios where dropouts preclude marker use or hierarchies are not established
- When only a bulk-average profile (single homogeneous population) is analysed with no matched single-cell reference — the bulk-average negative control is expected to be non-significant (P≈0.30)
- When the goal is trajectory/branch inference rather than potency scoring — the article notes this insight is not readily obtainable from Monocle or MPath and conversely pseudo-time is not SCENT's output

## Inputs

- Bulk RNA-Seq expression matrix for matched diagnostic–relapse AML samples (GEO-deposited)
- Single-cell RNA-Seq expression matrix for AML cells with CD34/CD96 marker annotations
- Single-cell RNA-Seq data for prostate circulating tumour cells from enzalutamide-treated and untreated patients
- Matched single-cell and bulk RNA-Seq expression matrices (e.g. melanoma cancer cells, T cells, B cells from 12 patients)
- Protein interaction network onto which genes are mapped for SCENT entropy estimation

## Outputs

- Per-sample bulk signalling entropy (SR) values and MRH z-scores with permutation P-values
- Per-cell SR rankings with marker-enrichment P-values for highest-entropy cells
- Comparison tables of SR differences and P-values, e.g. aml_ctc_scent_reproduction.csv and mrh_zscore_results.csv

## How to apply

Load expression data into R and map genes onto the protein interaction network required by SCENT; run SCENT's network-based signalling entropy (SR) algorithm on each single cell to build a reference single-cell SR distribution. Compute SR for each bulk sample and derive the MRH z-score as the standardized deviation of the bulk SR from that single-cell SR distribution, assessing significance with empirical/permutation P-values (negative control: bulk-average profile should be non-significant, P≈0.30). For phenotype comparisons, run paired tests within patients (e.g. paired Wilcoxon signed-rank test on diagnostic–relapse AML pairs, targeting P=0.004) or between-group tests (enzalutamide-treated vs untreated CTCs, targeting P=0.047), and rank single cells by entropy to test enrichment of markers such as CD34 (P=0.008) and CD96 (P=0.032). The rationale is that signalling entropy, computed in the context of an interaction network without feature selection, approximates differentiation potency more accurately than non-network entropy measures (StemID, SLICE) or pseudo-time methods (Monocle, MPath).

## Related tools

- **SCENT** (Computes network-based signalling entropy (SR) of single cells and bulk samples and estimates differentiation potency) — https://github.com/aet21/SCENT
- **CCAT** (Used alongside SCENT in the tutorial for estimating differentiation potency of single cells)
- **R** (Runtime environment in which SCENT is installed and the paired Wilcoxon, Fisher, and permutation tests are run)
- **BiocStyle** (Bioconductor styling package used for the SCENT/CCAT tutorial vignette)

## Examples

```
library(devtools); devtools::install_github("aet21/SCENT")
```

## Evaluation signals

- Relapsed AML bulk samples show significantly higher SR than matched diagnostic samples (paired Wilcoxon signed-rank test P=0.004)
- Highest-entropy single cells are enriched for CD34 (P=0.008) and CD96 (P=0.032)
- Enzalutamide-treated CTCs show a between-group SR difference vs untreated with P=0.047
- Melanoma bulk MRH z-scores significant across all 12 patients (combined Fisher test P<1e−6) while the bulk-average negative control is non-significant (P≈0.30)
- AML population SR (0.88) exceeds the maximum single-cell SR (0.82), consistent with regulated heterogeneity

## Limitations

- The precise relation between single-cell signalling entropy and transcriptional entropy (as defined by Richard et al.) is non-trivial and not established
- No existing assay can measure all single cells in a population, limiting validation of bulk-vs-single-cell synergy
- Criticality hallmarks such as an SI peak before a cell-fate transition remain untested pending high-resolution time-course and genome-wide scRNA-Seq data

## Evidence

- [abstract] signalling entropy allows in silico estimation of the differentiation potency and plasticity of single cells and bulk samples, providing a means to identify normal and cancer stem-cell phenotypes: "signalling entropy allows in silico estimation of the differentiation potency and plasticity of single cells and bulk samples, providing a means to identify normal and cancer stem-cell phenotypes"
- [abstract] Signalling entropy identifies known cell subpopulations of varying potency and drug resistant cancer stem-cell phenotypes, including those derived from circulating tumour cells: "Signalling entropy identifies known cell subpopulations of varying potency and drug resistant cancer stem-cell phenotypes, including those derived from circulating tumour cells"
- [methods] A brief tutorial for estimating differentiation potency of single cells using SCENT and CCAT: "A brief tutorial for estimating differentiation potency of single cells using SCENT and CCAT"
- [readme] The main purpose of the `SCENT` package is to provide a means of estimating the differentiation potency of single cells without the need to assume prior biological knowledge: "The main purpose of the `SCENT` package is to provide a means of estimating the differentiation potency of single cells without the need to assume prior biological knowledge"
- [discussion] The ability of signalling entropy to independently order single cells according to differentiation potency is a central component of the SCENT algorithm: "The ability of signalling entropy to independently order single cells according to differentiation potency is a central component of the SCENT algorithm"
