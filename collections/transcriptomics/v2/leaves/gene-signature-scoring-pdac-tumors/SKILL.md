---
name: gene-signature-scoring-pdac-tumors
description: Use when you have a bulk RNA-seq tumor expression matrix plus two competing
  gene signatures (basal-like vs classical) and need per-sample subtype calls comparable
  to published schema, followed by survival testing.
when_to_use_negative:
- Input is single-cell data needing subcluster discovery itself — use the full SCISSORS
  iterative reclustering routine instead of bulk signature scoring.
- You lack one of the two competing signatures (or only have one subtype's genes),
  so the higher-score-wins assignment rule cannot be applied.
- No survival metadata exists, making the Kaplan-Meier/log-rank validation step impossible.
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3518
tools:
- name: SCISSORS
  role: Source of the highly specific basal-like and classical marker genes that are
    fold-change ranked and used as scoring signatures on bulk tumors
  repo: https://github.com/jr-leary7/SCISSORS
- name: Seurat
  role: Single-cell infrastructure underlying the SCISSORS workflow that derives the
    signatures; also usable for signature scoring integration
- name: R
  role: Language for implementing signature scoring, concordance tabulation, and survival
    analysis
- name: cluster
  role: R clustering tools used in the SCISSORS workflow from which signatures derive
- name: ggplot2
  role: Plotting Kaplan-Meier survival curves and subtype visualizations
- name: dplyr
  role: Tidy manipulation of the tumor-by-signature score table and per-tumor call
    tabulation
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad449
    title: <u>S</u>ub-<u>C</u>luster <u>I</u>dentification through <u>S</u>emi-<u>S</u>upervised
      <u>O</u>ptimization of <u>R</u>are-Cell <u>S</u>ilhouettes (SCISSORS) in single-cell
      RNA-sequencing
  source_content_hash: sha256:174d31251385e0ae3ce7ab7827a24f3bfb0db1b28cf8f9ddcee082015d6e78e3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jr-leary7/SCISSORS
  tools:
  - R
  - SCISSORS
  - Seurat
  - cluster
  - dplyr
  - ggplot2
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# gene-signature-scoring-pdac-tumors

## Summary

Score bulk RNA-seq tumors (e.g., 150 TCGA PAAD samples) against fold-change-ranked SCISSORS-derived basal-like and classical PDAC gene signatures, assign each tumor the subtype of its higher-scoring signature, and validate calls against the Moffitt schema with Kaplan-Meier/log-rank survival analysis.

## When to use

Use when you have a bulk RNA-seq tumor expression matrix plus two competing gene signatures (basal-like vs classical) and need per-sample subtype calls comparable to published schema, followed by survival testing.

## When NOT to use

- Input is single-cell data needing subcluster discovery itself — use the full SCISSORS iterative reclustering routine instead of bulk signature scoring.
- You lack one of the two competing signatures (or only have one subtype's genes), so the higher-score-wins assignment rule cannot be applied.
- No survival metadata exists, making the Kaplan-Meier/log-rank validation step impossible.

## Inputs

- Bulk tumor RNA-seq expression matrix (e.g., TCGA PAAD, 150 tumors)
- Top-10 fold-change-ranked SCISSORS basal-like gene signature
- Top-10 fold-change-ranked SCISSORS classical gene signature
- Moffitt schema subtype calls per tumor (for concordance check)
- Survival metadata (time-to-event and status per patient)

## Outputs

- Per-tumor subtype calls (basal-like vs classical) with signature scores
- Concordance table vs Moffitt schema (e.g., 139/150 consistent; 33 basal-like, 106 classical)
- Kaplan-Meier survival curves by subtype with log-rank P-value and hazard ratio (e.g., P = .025, HR = 1.723)

## How to apply

Start from the top-10 fold-change-ranked SCISSORS basal-like and classical marker genes, anchored by their overlap with Moffitt genes. Score every tumor in the bulk matrix (e.g., 150 TCGA PAAD tumors) against both signatures and assign the subtype corresponding to the higher score, yielding per-tumor calls (target: 33 basal-like, 106 classical). Tabulate concordance with the Moffitt schema, expecting ~139-of-150 consistent calls. Fit Kaplan-Meier curves for basal-like versus classical groups and evaluate separation with the log-rank test; the reference result is P = .025 with hazard ratio = 1.723 for shorter basal-like survival (Fig. 5k). Rationale: SCISSORS provides a method for identification of highly specific marker genes from single-cell subclusters, which can be projected onto bulk tumors to reproduce published subtyping de novo.

## Related tools

- **SCISSORS** (Source of the highly specific basal-like and classical marker genes that are fold-change ranked and used as scoring signatures on bulk tumors) — https://github.com/jr-leary7/SCISSORS
- **Seurat** (Single-cell infrastructure underlying the SCISSORS workflow that derives the signatures; also usable for signature scoring integration)
- **R** (Language for implementing signature scoring, concordance tabulation, and survival analysis)
- **cluster** (R clustering tools used in the SCISSORS workflow from which signatures derive)
- **ggplot2** (Plotting Kaplan-Meier survival curves and subtype visualizations)
- **dplyr** (Tidy manipulation of the tumor-by-signature score table and per-tumor call tabulation)

## Examples

```
library(dplyr); library(Seurat); library(ggplot2); scores <- sapply(list(basal_sig, classical_sig), function(g) colMeans(expr[rownames(expr) %in% g, ])); subtype <- apply(scores, 1, which.max)
```

## Evaluation signals

- Concordance with Moffitt schema reaches ~139/150 consistent calls (33 basal-like, 106 classical) on TCGA PAAD.
- Log-rank test on basal-like vs classical Kaplan-Meier curves yields P = .025 with hazard ratio = 1.723, basal-like showing shorter survival.
- Each tumor receives exactly one subtype call, determined by the higher-scoring of the two signatures.
- Signatures are anchored by their overlap with published Moffitt genes, confirming signature identity before scoring.
- Signature genes are the top-10 by fold change from SCISSORS marker-gene identification, not arbitrary selections.

## Limitations

- Even small differences in analysis and parameter choice can greatly alter scRNA-seq clustering results, which affects the signatures that feed this scoring step.
- Validation result (139/150 concordance, P = .025, HR = 1.723) is specific to TCGA PAAD bulk data; other cohorts may differ.
- No discussion section text was available in the provided document, so reproducibility gaps or limitations beyond those stated could not be extracted.

## Evidence

- [results] Signature ranking rule: "patients with basal-like tumors subtyped by SCISSORS genes showed significantly shorter survival (P = .025, hazard ratio = 1.723, and log-rank test)"
- [abstract] SCISSORS marker specificity: "Additionally, SCISSORS provides a method for the identification of marker genes of high specificity to the cell type"
- [abstract] Parameter sensitivity caveat: "even small differences in analysis and parameter choosing can greatly alter clustering results and thus impose great influence on which cell types are identified"
- [methods] R toolchain: "library(dplyr)           # tidy data 
library(Seurat)          # single cell infrastructure"
- [readme] Package install: "You can install the most recent version of `SCISSORS` with:

 r
remotes::install_github("jr-leary7/SCISSORS")
"
