---
name: signalling-entropy-potency-estimation
description: Use when scRNA-seq profiles must be ordered by differentiation potency
  without markers or timepoints — e.g. snapshot data of complex tissues, high-dropout
  datasets, or identifying cancer stem-cell/drug-resistant phenotypes.
when_to_use_negative:
- When prior biological knowledge such as marker expression or timepoint labels is
  reliable and preferred — SCENT is intended for cases where such knowledge is unavailable
  or dropouts preclude marker use
- When the goal is trajectory/pseudo-time inference by clustering or branch analysis
  — the article notes the insight is 'not readily obtainable using other algorithms
  such as Monocle or MPath', but conversely Monocle/MPath-style tools serve different
  purposes
- When only transcriptomic (non-network) entropy is required — StemID/SLICE-style
  measures differ because they do not use network information and are less accurate
  here
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_3308
tools:
- name: SCENT
  role: Core R package that computes single-cell and bulk signalling entropy, fits
    mixture-of-Gaussians potency states, and provides the PPI connectome
  repo: https://github.com/aet21/SCENT
- name: R
  role: Execution environment for the SCENT package and downstream statistical tests
    (Wilcoxon, Spearman correlation)
- name: BiocStyle
  role: Bioconductor styling used in the SCENT tutorial vignette/documentation
- name: CCAT
  role: Companion tool used alongside SCENT in the tutorial for estimating differentiation
    potency of single cells
provenance:
  source_task_ids:
  - task_004
  - task_006
  - task_001
  source_papers:
  - doi: 10.1038/ncomms15599
    title: Single-cell entropy for accurate estimation of differentiation potency
      from a cell’s transcriptome
  source_content_hash: sha256:7e9deb34f66c2285feaf7ff2af503531e92dc5881094d3903afa5aacabceec4b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/aet21/SCENT
  tools:
  - BiocStyle
  - CCAT
  - R
  - SCENT
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0602
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# signalling-entropy-potency-estimation

## Summary

Estimates the differentiation potency of individual cells by computing signalling entropy (SR): the signalling promiscuity of a cell's transcriptome integrated with a protein-protein interaction (connectome) network, implemented in the SCENT R package. Requires no feature selection, marker genes, timepoint labels, or prior biological knowledge.

## When to use

Use when scRNA-seq profiles must be ordered by differentiation potency without markers or timepoints — e.g. snapshot data of complex tissues, high-dropout datasets, or identifying cancer stem-cell/drug-resistant phenotypes.

## When NOT to use

- When prior biological knowledge such as marker expression or timepoint labels is reliable and preferred — SCENT is intended for cases where such knowledge is unavailable or dropouts preclude marker use
- When the goal is trajectory/pseudo-time inference by clustering or branch analysis — the article notes the insight is 'not readily obtainable using other algorithms such as Monocle or MPath', but conversely Monocle/MPath-style tools serve different purposes
- When only transcriptomic (non-network) entropy is required — StemID/SLICE-style measures differ because they do not use network information and are less accurate here

## Inputs

- Single-cell RNA-seq expression matrix (genes x cells), e.g. Chu et al. hESC/progenitor data (1,018 cells)
- Human protein-protein interaction (connectome) network supplied with SCENT
- Bulk RNA-seq expression data (for SCENT bulk SR estimation, e.g. paired diagnostic-relapse AML samples)
- Optional cell class labels or marker information (CD34/CD96) for validation only

## Outputs

- Per-cell normalized signalling entropy rate (SR) values, optionally on logit-SR scale
- Per-cell discrete potency-state assignments from mixture-of-Gaussians fitting
- Per-population Shannon index (SI) of potency heterogeneity
- Group-level SR comparisons with statistical test statistics and P-values (e.g. paired Wilcoxon)
- CSV files of per-cell SR values and potency states (e.g. chu_scent_sr.csv, scent_chu_results.csv)

## How to apply

Install SCENT from GitHub (aet21/SCENT) in R with BiocStyle, then map the input expression matrix (e.g. the Chu et al. hESC/NPC matrix of 1,018 cells) genes onto the SCENT signalling interaction-network (connectome) dataset. Compute the normalized single-cell signalling entropy rate (SR) for each cell by integrating expression with the PPI network, optionally transforming to logit-SR scale. Fit a mixture-of-Gaussians model to the logit-SR values to assign cells to discrete potency states and verify parsimony (e.g. a two-state solution with ~4% of hESCs in the low-potency state). Optionally estimate a Shannon index (SI) of potency heterogeneity over the population from the distribution of potency states. For bulk RNA-seq, use SCENT's bulk estimation mode; SR comparisons between groups (e.g. paired diagnostic-relapse AML) are tested with paired Wilcoxon signed-rank tests. Write per-cell SR values, potency-state assignments, and per-population Shannon indices to a CSV output.

## Related tools

- **SCENT** (Core R package that computes single-cell and bulk signalling entropy, fits mixture-of-Gaussians potency states, and provides the PPI connectome) — https://github.com/aet21/SCENT
- **R** (Execution environment for the SCENT package and downstream statistical tests (Wilcoxon, Spearman correlation))
- **BiocStyle** (Bioconductor styling used in the SCENT tutorial vignette/documentation)
- **CCAT** (Companion tool used alongside SCENT in the tutorial for estimating differentiation potency of single cells)

## Examples

```
library(devtools); devtools::install_github("aet21/SCENT"); library(SCENT)
```

## Evaluation signals

- On Chu et al. data, SR should discriminate pluripotent hESCs from non-pluripotent progenitors with ROC AUC = 0.96 and Spearman correlation with pluripotency score of 0.91 (Wilcoxon P<1e-300)
- Mixture-of-Gaussians potency-state inference should recover a parsimonious two-state model, with ~4% of hESCs assigned to the low-potency state
- The NPC population should attain the highest Shannon index of potency heterogeneity across populations
- On paired AML diagnostic-relapse bulk samples, relapsed samples should show higher SR (paired Wilcoxon P=0.004); CD34 and CD96 enriched among highest-entropy single cells (P=0.008 and P=0.032)
- Enzalutamide-treated CTCs vs untreated should yield a between-group SR difference with P=0.047

## Limitations

- The precise relation between single-cell signalling entropy and transcriptional entropy (as defined by Richard et al.) is non-trivial and not established
- Criticality hallmarks (e.g. an SI peak before a cell-fate transition) remain untested, pending high-resolution time-course and genome-wide scRNA-seq data
- No existing assay can measure all single cells in a population, limiting validation of bulk-vs-single-cell synergy
- Accuracy depends on integrating the transcriptome with a PPI network; transcriptomic entropy measures without network information are less accurate

## Evidence

- [abstract] differentiation potency of a single cell can be approximated by computing the signalling promiscuity, or entropy, of a cell's transcriptome in the context of an interaction network: "differentiation potency of a single cell can be approximated by computing the signalling promiscuity, or entropy, of a cell's transcriptome in the context of an interaction network"
- [methods] The main purpose of the `SCENT` package is to provide a means of estimating the differentiation potency of single cells without the need to assume prior biological knowledge: "The main purpose of the `SCENT` package is to provide a means of estimating the differentiation potency of single cells without the need to assume prior biological knowledge"
- [discussion] we can assign single cells into potency states, from which a SI over the whole cell population (that is, using the distribution of potency states over single cells) can then be estimated: "we can assign single cells into potency states, from which a SI over the whole cell population (that is, using the distribution of potency states over single cells) can then be estimated"
- [abstract] Signalling entropy provides a more accurate and robust potency estimate than other entropy-based measures, driven in part by a subtle positive correlation between the transcriptome and connectome: [evidence span withheld — anchor status: not_found]
- [readme] devtools::install_github("aet21/SCENT"): "devtools::install_github("aet21/SCENT")"
