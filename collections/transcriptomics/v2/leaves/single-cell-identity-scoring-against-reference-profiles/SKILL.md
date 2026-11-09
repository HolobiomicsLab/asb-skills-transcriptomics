---
name: single-cell-identity-scoring-against-reference-profiles
description: Use when single-cell RNA-seq data from a reprogramming or fate-conversion
  experiment must be scored per cell for source (fibroblast) versus target (iEP) identity
  to compare conditions, e.g. testing whether adding Fos/Yap1 to a cocktail increases
  target identity scores.
when_to_use_negative:
- The data are not single-cell gene expression profiles, e.g. bulk RNA-seq or GRN
  edge lists — Capybara QP classification requires a genes x cells matrix
- The goal is to simulate perturbation effects on identity rather than score observed
  identity; use CellOracle's in silico KO/overexpression simulation instead
- No annotated reference profiles exist for the source and target cell states, so
  fibroblast/iEP scores cannot be anchored
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: Capybara
  role: Performs quadratic-programming-based cell type classification and continuum
    identity scoring of day-20 cells against fibroblast and iEP reference profiles
  repo: https://github.com/morris-lab/Capybara
- name: Cell Ranger 6.0.1
  role: Produces the processed 10x Chromium scRNA-seq count matrix that serves as
    the classification input
- name: R
  role: Runtime environment for Capybara (requires R >= 3.5.0); dependencies installed
    via CRAN/Bioconductor (limma, impute, AnnotationDbi, GO.db, preprocessCore)
- name: igraph
  role: R dependency checked during CellOracle/Capybara-adjacent library installation
  repo: https://github.com/igraph/rigraph
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.stemcr.2022.11.010
    title: Gene regulatory network reconfiguration in direct lineage reprogramming
  source_content_hash: sha256:e9eb588699a6d090661554bd8dc56cd47b64bbde4082824939fa63be06823a46
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/morris-lab/CellOracle
  repo_url: https://github.com/morris-lab/Capybara
  tools:
  - Capybara
  - Cell Ranger
  - Cell Ranger 6.0.1
  - R
  - igraph
  edam_operation: http://edamontology.org/operation_3803
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# single-cell-identity-scoring-against-reference-profiles

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Scores individual cells for fibroblast versus induced endoderm progenitor (iEP) identity by classifying them against reference transcriptomic profiles with Capybara's quadratic-programming-based cell type classification, enabling quantitative comparison of identity across reprogramming cocktails (e.g. Hnf4α-Foxa1 with or without Fos/Yap1).

## When to use

Use when single-cell RNA-seq data from a reprogramming or fate-conversion experiment must be scored per cell for source (fibroblast) versus target (iEP) identity to compare conditions, e.g. testing whether adding Fos/Yap1 to a cocktail increases target identity scores.

## When NOT to use

- The data are not single-cell gene expression profiles, e.g. bulk RNA-seq or GRN edge lists — Capybara QP classification requires a genes x cells matrix
- The goal is to simulate perturbation effects on identity rather than score observed identity; use CellOracle's in silico KO/overexpression simulation instead
- No annotated reference profiles exist for the source and target cell states, so fibroblast/iEP scores cannot be anchored

## Inputs

- Cell Ranger v6.0.1-processed day-20 Perturb-seq scRNA-seq count matrix (genes x cells)
- Single-cell reference profiles with cell-type annotations (e.g. a cell atlas in matrix form with metadata mapping barcodes to cell.type)
- Bulk reference transcriptome matrix (e.g. Capybara's ARCHS4-derived C57BL/6 bulk raw counts/RPKM)
- Per-cell metadata with reprogramming cocktail assignment (Hnf4α-Foxa1 +/- Fos +/- Yap1)

## Outputs

- Per-cell fibroblast and iEP identity scores (continuum-valued classification)
- Per-condition aggregated identity scores (identity_scores.csv)

## How to apply

Load the Cell Ranger v6.0.1-processed day-20 Perturb-seq scRNA-seq count matrix, then run Capybara, which measures cell identity as a continuum at single-cell resolution by applying quadratic programming against bulk and single-cell reference profiles rather than forcing discrete assignment. Score each day-20 cell for fibroblast and iEP identity against reference profiles, allowing cells with mixed identities to receive fractional scores. Aggregate per-cell identity scores by reprogramming cocktail (Hnf4α-Foxa1 with/without Fos and/or Yap1) and export per-cell and per-condition summaries as identity_scores.csv. Rationale: Capybara was chosen by the authors precisely because reprogramming populations contain intermediate states, so continuum scoring outperforms hard clustering for comparing conversion efficiency across conditions.

## Related tools

- **Capybara** (Performs quadratic-programming-based cell type classification and continuum identity scoring of day-20 cells against fibroblast and iEP reference profiles) — https://github.com/morris-lab/Capybara
- **Cell Ranger 6.0.1** (Produces the processed 10x Chromium scRNA-seq count matrix that serves as the classification input)
- **R** (Runtime environment for Capybara (requires R >= 3.5.0); dependencies installed via CRAN/Bioconductor (limma, impute, AnnotationDbi, GO.db, preprocessCore))
- **igraph** (R dependency checked during CellOracle/Capybara-adjacent library installation) — https://github.com/igraph/rigraph

## Examples

```
library("devtools"); devtools::install_github("morris-lab/Capybara"); library("Capybara"); single.round.QP.analysis(bulk.raw, sc.expr, scale.bulk.sc = "scale", unix.par = TRUE, force.eq = 1, n.cores = 4, save.to.path = "./", save.to.filename = "identity_scores")
```

## Evaluation signals

- identity_scores.csv exists with both per-cell rows and per-condition aggregates and is parseable as a CSV
- The Hnf4α-Foxa1 + Fos + Yap1 condition shows higher iEP identity scores than the cocktail lacking Fos/Yap1, matching the paper's reported reprogramming efficiencies
- Per-cell scores are continuum-valued (not hard 0/1 assignments), consistent with Capybara's design to measure identity at single-cell resolution
- Scored cells correspond 1:1 with barcodes in the Cell Ranger v6.0.1 day-20 Perturb-seq matrix (no dropped or duplicated cells)
- Reference loading succeeds with cell.type/barcode metadata columns intact (Capybara expects meta with cell.type and barcode columns)

## Limitations

- Capybara requires R >= 3.5.0 and manual Bioconductor installation of limma, impute, AnnotationDbi, GO.db, and preprocessCore
- Results can vary with threshold selection; the Capybara reproducibility notebooks note 'There could exist some variations due to difference in threshold selections'
- Large single-cell reference count matrices are not bundled with the package and must be downloaded separately (e.g. MCA from figshare), and Windows users must set unix.par=F and n.cores=1
- The iEPs-resemble-post-injury-BECs interpretation is attributed to a parallel study (Kong et al., 2022), so identity scoring alone does not establish regenerative identity

## Evidence

- [other] Score each day-20 cell for fibroblast and iEP identity against reference profiles using Capybara-style cell type classification.: [evidence span withheld — anchor status: not_found]
- [readme] Capybara is a tool to measure cell identity and fate transitions... at a single-cell resolution.: [evidence span withheld — anchor status: external_doc]
- [other] confirm... that the Hnf4α-Foxa1 + Fos + Yap1 condition shows higher iEP identity scores than the cocktail lacking Fos/Yap1, consistent with the paper's reported reprogramming efficiencies.: [evidence span withheld — anchor status: not_found]
