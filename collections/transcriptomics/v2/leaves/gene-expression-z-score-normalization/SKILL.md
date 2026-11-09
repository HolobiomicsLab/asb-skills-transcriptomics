---
name: gene-expression-z-score-normalization
description: Use when raw perturbation-series microarray data must be normalized per
  series and converted to z-scores so expression changes from different perturbation
  experiments (e.g. ArrayExpress submissions) can be pooled for signature construction.
when_to_use_negative:
- Input data is already z-scored or otherwise normalized to a common scale — re-z-scoring
  would remove the signal you need
- You only want to score pathway activity in new bulk/single-cell samples using the
  published PROGENy signatures (use progeny/decoupler scoring, not this construction
  step)
- The perturbation experiments target pathways outside the 11 modeled pathways (EGFR,
  Hypoxia, JAK-STAT, MAPK, NFkB, p53/DDR, PI3K, TGFb, Trail, VEGF/PDGF), since the
  binary perturbation indicator model is defined only for these
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3518
tools:
- name: progeny
  role: R package that ships the pathway signatures produced downstream of this z-score
    normalization step
  repo: github.com/saezlab/progeny
- name: footprints
  role: Repository containing the normalize_data.r, expr.r, and zscores.r scripts
    that implement the normalization and z-score computation
  repo: saezlab/footprints
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-017-02391-6
    title: Perturbation-response genes reveal signaling footprints in cancer gene
      expression
  source_content_hash: sha256:2961cc755635f5e9eec4a9afb90b7d6e539b47acd9ea5afd97f99f0baeedb19d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - footprints
  - knitr
  - progeny
  - rmarkdown
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# gene-expression-z-score-normalization

## Summary

Per-experiment z-score transformation of microarray gene-expression data from signaling perturbation experiments, the normalization step PROGENy uses to remove experiment-level effects before pooling samples across ~580 perturbation experiments covering 11 pathways.

## When to use

Use when raw perturbation-series microarray data must be normalized per series and converted to z-scores so expression changes from different perturbation experiments (e.g. ArrayExpress submissions) can be pooled for signature construction.

## When NOT to use

- Input data is already z-scored or otherwise normalized to a common scale — re-z-scoring would remove the signal you need
- You only want to score pathway activity in new bulk/single-cell samples using the published PROGENy signatures (use progeny/decoupler scoring, not this construction step)
- The perturbation experiments target pathways outside the 11 modeled pathways (EGFR, Hypoxia, JAK-STAT, MAPK, NFkB, p53/DDR, PI3K, TGFb, Trail, VEGF/PDGF), since the binary perturbation indicator model is defined only for these

## Inputs

- Raw/normalized microarray gene-expression data for perturbation series from ArrayExpress (208 submissions, 580 experiments, 11 pathways)
- YAML-format experiment index files with search terms (query.txt) and .excluded annotations for failed-QC or ambiguous perturbations

## Outputs

- Per-experiment z-scored gene-expression matrices (zscores.r output)
- Assembled expression matrix ready for gene-wise linear modeling (input to speed_matrix model construction)

## How to apply

Treat each perturbation series as a whole: first normalize and QC the entire series (scripts normalize_data.r and expr.r in the footprints data directory), excluding series or arrays that fail QC or where the perturbation does not correspond to the desired phenotype. Then compute z-scores of gene expression for each individual perturbation experiment (zscores.r), which centers and scales expression so experiment-level effects are removed. Before pooling z-scores across experiments, PROGENy fits a LOESS-smoothed null model to the z-scored expression to account for the mean-variance relationship. The rationale is that fold-changes are heterogeneous and condition-dependent, whereas the common core of pathway-responsive genes only emerges once each experiment is placed on a common z-score scale. These z-scores then serve as the response variable in a gene-wise linear regression against a binary matrix of pathway perturbation indicators.

## Related tools

- **progeny** (R package that ships the pathway signatures produced downstream of this z-score normalization step) — github.com/saezlab/progeny
- **footprints** (Repository containing the normalize_data.r, expr.r, and zscores.r scripts that implement the normalization and z-score computation) — saezlab/footprints

## Examples

```
Rscript data/normalize_data.r && Rscript data/zscores.r
```

## Evaluation signals

- Each perturbation experiment's z-scored expression should have per-gene means centered near zero and unit-variance scaling across samples
- Experiments excluded for QC failure or ambiguous perturbation phenotype must be flagged via the .excluded suffix or a commented entry in the index file, with the reason stated
- The z-score matrix must align rows (genes) with the binary pathway-perturbation indicator matrix used as the independent variable in the linear model
- After LOESS null-model correction, the pooled z-scores should show the mean-variance relationship accounted for before gene-wise regression
- Downstream sanity check: the regression on the z-scores should recover the top 100 non-zero responsive genes per pathway in the 11 x 100 speed_matrix model

## Limitations

- Experiments are dropped when QC fails or the perturbation does not correspond to the phenotype of interest; these are only marked via .excluded annotations, so silent inclusion of bad series is a risk
- Individual perturbation signatures are heterogeneous; z-score pooling captures the common core, but additional transcriptional modules may not always be activated
- Tissue-specific signatures may reflect pathway activation more closely than the consensus signatures derived from pooled z-scores in any single context

## Evidence

- [readme] per-experiment z-score computation scripts: [evidence span withheld — anchor status: external_doc]
- [readme] z-scores as regression input: [evidence span withheld — anchor status: external_doc]
- [readme] exclusion annotation convention: [evidence span withheld — anchor status: external_doc]
- [abstract] leveraging perturbation compendium: "Here we present PROGENy, a method that overcomes both limitations by leveraging a large compendium of publicly available perturbation experiments"
- [readme] common core despite heterogeneity: [evidence span withheld — anchor status: external_doc]
