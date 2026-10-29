---
name: arrayexpress-perturbation-data-curation
description: 'Use when building consensus pathway-response signatures from public
  perturbation data: ArrayExpress series must be searched, pathway-labeled, QC''d,
  and z-scored per experiment before signature fitting.'
when_to_use_negative:
- Input is already a z-scored perturbation compendium or a finished pathway signature
  matrix — curation is complete and this skill would duplicate work
- The candidate perturbation does not clearly correspond to the pathway phenotype
  to be observed — the README states such files are excluded rather than curated in
- Data are observational (e.g., TCGA primary tumors or GDSC cell lines) rather than
  designed perturbation experiments — those need pathway-scoring methods (PROGENy
  scores, GSVA, SPIA, Pathifier, PARADIGM), not compendium curation
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: progeny
  role: R/Bioconductor package that stores the pathway signatures ultimately produced
    from the curated perturbation compendium; the curation workflow supplies its 11
    x 100 gene-by-pathway coefficient matrix
  repo: github.com/saezlab/progeny
- name: footprints
  role: Analysis repository hosting the index/ YAML curation files and the data/ scripts
    (normalize_data.r, expr.r, zscores.r) that implement the QC and z-scoring steps
  repo: github.com/saezlab/footprints
- name: rmarkdown
  role: Vignette/reporting engine used to render the methods documentation accompanying
    the workflow
  repo: github.com/saezlab/progeny
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-017-02391-6
    title: Perturbation-response genes reveal signaling footprints in cancer gene
      expression
  source_content_hash: sha256:340861a33e1f0d10231d4b12924f72b59595538d4101406048b3fa8d6b5e7315
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - footprints
  - knitr
  - progeny
  - rmarkdown
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6470588235294118
n_claims: 17
---

# arrayexpress-perturbation-data-curation

## Summary

Curation workflow that retrieves, QC-filters, and pathway-labels public perturbation gene-expression experiments — 208 ArrayExpress submissions totaling 580 experiments across 11 PROGENy signaling pathways — and converts them into per-experiment z-scores that feed the linear-model construction of pathway-responsive gene signatures.

## When to use

Use when building consensus pathway-response signatures from public perturbation data: ArrayExpress series must be searched, pathway-labeled, QC'd, and z-scored per experiment before signature fitting.

## When NOT to use

- Input is already a z-scored perturbation compendium or a finished pathway signature matrix — curation is complete and this skill would duplicate work
- The candidate perturbation does not clearly correspond to the pathway phenotype to be observed — the README states such files are excluded rather than curated in
- Data are observational (e.g., TCGA primary tumors or GDSC cell lines) rather than designed perturbation experiments — those need pathway-scoring methods (PROGENy scores, GSVA, SPIA, Pathifier, PARADIGM), not compendium curation

## Inputs

- ArrayExpress search terms (query.txt) per pathway for the 11 PROGENy signaling pathways
- YAML-format index files listing perturbation submissions and experiments per pathway
- Raw/normalized microarray gene-expression series downloaded from ArrayExpress (208 submissions, 580 experiments)
- Perturbation annotations mapping each experiment to the perturbed pathway(s), including multi-pathway perturbations

## Outputs

- Curated YAML experiment index with included/excluded entries and exclusion reasons
- Whole-series normalized and QC'd gene-expression data
- Per-experiment z-scored gene-expression matrix across the perturbation compendium
- Binary pathway-perturbation indicator matrix aligned to the z-scores (input to the speed_matrix linear model)

## How to apply

For each of the 11 pathways (EGFR, Hypoxia, JAK-STAT, MAPK, NFkB, p53/DDR, PI3K, TGFb, Trail, VEGF/PDGF), search ArrayExpress using per-pathway query.txt search terms and record the matching submissions in a YAML index directory, noting 208 submissions / 580 experiments. Mark files and series excluded because of QC failure or uncertain phenotype-perturbation correspondence with a '.excluded' suffix or a commented entry stating the reason. Normalize and QC each series as a whole (normalize_data.r), assemble the relevant expression data (expr.r), then compute z-scores of expression for each perturbation experiment (zscores.r) to remove experiment-level effects. The rationale is that individual perturbation signatures are heterogeneous, so a curated, QC-filtered compendium is needed to expose a common core of pathway-responsive genes. The resulting z-score matrix plus a binary pathway-perturbation indicator matrix (including multi-pathway perturbations) is what the downstream linear model (speed_matrix) consumes.

## Related tools

- **progeny** (R/Bioconductor package that stores the pathway signatures ultimately produced from the curated perturbation compendium; the curation workflow supplies its 11 x 100 gene-by-pathway coefficient matrix) — github.com/saezlab/progeny
- **footprints** (Analysis repository hosting the index/ YAML curation files and the data/ scripts (normalize_data.r, expr.r, zscores.r) that implement the QC and z-scoring steps) — github.com/saezlab/footprints
- **rmarkdown** (Vignette/reporting engine used to render the methods documentation accompanying the workflow) — github.com/saezlab/progeny

## Examples

```
Rscript data/normalize_data.r && Rscript data/expr.r && Rscript data/zscores.r
```

## Evaluation signals

- Index covers 208 ArrayExpress submissions yielding 580 experiments across the 11 pathways; counts can be reconciled against the YAML index directory
- Every excluded file or experiment carries a '.excluded' suffix or a commented entry in the index stating the exclusion reason
- Z-scores are computed per perturbation experiment after whole-series normalization and QC, not pooled before normalization
- The binary perturbation-indicator matrix has the same experiment dimension as the z-score matrix and marks multi-pathway perturbations correctly
- Downstream linear model (speed_matrix) selects the 100 most significant genes per pathway, matching the published 11 x 100 signature dimensions

## Limitations

- Exclusion decisions are judgment-based: series are dropped for QC failure or when 'we were not sure if the perturbation corresponds to the phenotype we want to observe'
- Individual perturbation signatures are heterogeneous; the curation targets only the common core of responsive genes, and tissue-specific signatures may more closely reflect pathway activation in a specific context
- The compendium covers 11 specified signaling pathways and human perturbation data; pathways without public perturbation experiments (e.g., directed pathway resources) fall outside its scope

## Evidence

- [readme] compendium scale: [evidence span withheld — anchor status: external_doc]
- [readme] exclusion policy: [evidence span withheld — anchor status: external_doc]
- [readme] normalization and z-scoring scripts: [evidence span withheld — anchor status: external_doc]
- [readme] downstream model input: [evidence span withheld — anchor status: external_doc]
- [abstract] rationale for compendium curation: "leveraging a large compendium of publicly available perturbation experiments to yield a common core of Pathway RespOnsive GENes"
