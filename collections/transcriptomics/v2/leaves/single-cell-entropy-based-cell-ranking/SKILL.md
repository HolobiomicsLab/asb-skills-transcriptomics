---
name: single-cell-entropy-based-cell-ranking
description: Use when you have scRNA-Seq or bulk RNA-Seq expression data and need
  to rank cells/samples by differentiation potency or detect cancer stem-cell/drug-resistant
  phenotypes without marker genes or timepoint labels.
when_to_use_negative:
- When prior biological knowledge such as marker expression or timepoint is reliable
  and sufficient — the README states SCENT's purpose is to estimate potency 'without
  the need to assume prior biological knowledge', so it is unnecessary when such priors
  are already definitive.
- When a purely trajectory-based pseudo-time method suffices — the article notes the
  key insight 'is not readily obtainable using other algorithms such as Monocle or
  MPath', but conversely Monocle/MPath may be preferred when the goal is trajectory
  inference rather than potency estimation.
- When the input is already a potency/differentiation-state assignment rather than
  raw expression — SCENT computes entropy from the transcriptome in the context of
  an interaction network and cannot run on precomputed feature tables.
edam_operation: http://edamontology.org/operation_0533
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0605
tools:
- name: SCENT
  role: Core R package that computes signalling entropy (SR) from single-cell or bulk
    expression data to estimate differentiation potency and rank cells
  repo: https://github.com/aet21/SCENT
- name: CCAT
  role: Used together with SCENT in the tutorial workflow for estimating differentiation
    potency of single cells
- name: R
  role: Runtime environment for the SCENT/CCAT tutorial and downstream statistical
    tests (Wilcoxon signed-rank, enrichment tests)
- name: BiocStyle
  role: Bioconductor styling package used to render the SCENT tutorial documentation
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/ncomms15599
    title: Single-cell entropy for accurate estimation of differentiation potency
      from a cell’s transcriptome
  source_content_hash: sha256:9fa7148544c49048395a5aa3cffa246b453e810a38a733cb342d102257644f53
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/aet21/SCENT
  tools:
  - BiocStyle
  - CCAT
  - R
  - SCENT
  edam_operation: http://edamontology.org/operation_0533
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# single-cell-entropy-based-cell-ranking

## Summary

Estimate the differentiation potency of single cells (or bulk samples) by computing signalling entropy (SR) — signalling promiscuity of a cell's transcriptome in the context of a protein interaction network — using the SCENT R package, in order to rank cells by potency and identify stem-like or drug-resistant subpopulations.

## When to use

Use when you have scRNA-Seq or bulk RNA-Seq expression data and need to rank cells/samples by differentiation potency or detect cancer stem-cell/drug-resistant phenotypes without marker genes or timepoint labels.

## When NOT to use

- When prior biological knowledge such as marker expression or timepoint is reliable and sufficient — the README states SCENT's purpose is to estimate potency 'without the need to assume prior biological knowledge', so it is unnecessary when such priors are already definitive.
- When a purely trajectory-based pseudo-time method suffices — the article notes the key insight 'is not readily obtainable using other algorithms such as Monocle or MPath', but conversely Monocle/MPath may be preferred when the goal is trajectory inference rather than potency estimation.
- When the input is already a potency/differentiation-state assignment rather than raw expression — SCENT computes entropy from the transcriptome in the context of an interaction network and cannot run on precomputed feature tables.

## Inputs

- single-cell RNA-Seq expression matrix (e.g. AML scRNA-Seq with CD34/CD96 markers, prostate CTC scRNA-Seq from enzalutamide-treated and untreated patients)
- bulk RNA-Seq expression matrix (e.g. GEO-deposited paired diagnostic–relapse AML samples)
- protein interaction network (connectome) required by SCENT
- optional marker/gene-set annotations (e.g. Molecular Signatures Database terms)

## Outputs

- per-cell or per-sample signalling entropy (SR) values and potency ranking
- potency-state assignments and a population-level stemness index (SI)
- group-comparison statistics (e.g. paired Wilcoxon P=0.004 for relapsed vs diagnostic AML; CTC P=0.047; CD34/CD96 enrichment P=0.008 and P=0.032)
- compiled results table (e.g. aml_ctc_scent_reproduction.csv)

## How to apply

Install SCENT from GitHub (devtools::install_github("aet21/SCENT")) and run its entropy estimator on a single-cell expression matrix together with a protein interaction network, computing the signalling entropy (SR) per cell without prior feature selection. Rank cells by SR to order them along potency; for paired bulk data (e.g. diagnostic–relapse AML), use SCENT's bulk estimation mode and compare within-patient SR values with a paired Wilcoxon signed-rank test. For phenotype association (e.g. CD34/CD96 enrichment in AML, or enzalutamide-treated vs untreated CTCs), test enrichment of markers among the highest-entropy cells or between-group SR differences. Optionally use CCAT alongside SCENT for potency-state assignment, from which a stemness index (SI) over the whole cell population can be estimated from the distribution of potency states. Rationale: signalling entropy exploits a subtle positive correlation between transcriptome and connectome, making it more accurate and robust than purely transcriptomic entropy measures (StemID, SLICE) and applicable without biological priors.

## Related tools

- **SCENT** (Core R package that computes signalling entropy (SR) from single-cell or bulk expression data to estimate differentiation potency and rank cells) — https://github.com/aet21/SCENT
- **CCAT** (Used together with SCENT in the tutorial workflow for estimating differentiation potency of single cells)
- **R** (Runtime environment for the SCENT/CCAT tutorial and downstream statistical tests (Wilcoxon signed-rank, enrichment tests))
- **BiocStyle** (Bioconductor styling package used to render the SCENT tutorial documentation)

## Examples

```
library(devtools); devtools::install_github("aet21/SCENT"); library(SCENT)
```

## Evaluation signals

- Reproducing the paired Wilcoxon signed-rank test on diagnostic–relapse AML bulk SR values yields P=0.004, with relapsed samples showing higher entropy.
- Enrichment of CD34 and CD96 among the highest-entropy AML single cells gives the reported P=0.008 and P=0.032.
- Between-group SR comparison of enzalutamide-treated vs untreated CTCs reproduces P=0.047.
- Per-cell SR values rank cells consistently with known potency hierarchies (e.g. pluripotent > multipotent > differentiated) without feature selection or training.
- Output table (e.g. aml_ctc_scent_reproduction.csv) contains SR values and P-values matching the reported comparisons.

## Limitations

- The precise relation between single-cell signalling entropy and transcriptional entropy (as defined by Richard et al.) is non-trivial and not established.
- No existing assay can measure all single cells in a population, limiting validation of bulk-vs-single-cell synergy.
- Criticality hallmarks (e.g. an SI peak before a cell-fate transition) remain untested pending high-resolution time-course and genome-wide scRNA-Seq data.
- Enriched biological terms are only reported when they achieve a significant adjusted P-value and OR>5.

## Evidence

- [readme] Potency without priors: "estimating the differentiation potency of single cells without the need to assume prior biological knowledge such as marker expression or timepoint"
- [abstract] Abstract claim: "differentiation potency of a single cell can be approximated by computing the signalling promiscuity, or entropy, of a cell's transcriptome in the context of an interaction network"
- [abstract] Drug-resistant phenotypes: "Signalling entropy identifies known cell subpopulations of varying potency and drug resistant cancer stem-cell phenotypes, including those derived from circulating tumour cells"
- [methods] Tutorial workflow: "A brief tutorial for estimating differentiation potency of single cells using SCENT and CCAT"
- [abstract] Robustness rationale: [evidence span withheld — anchor status: not_found]
- [discussion] Population-level index: "we can assign single cells into potency states, from which a SI over the whole cell population"
