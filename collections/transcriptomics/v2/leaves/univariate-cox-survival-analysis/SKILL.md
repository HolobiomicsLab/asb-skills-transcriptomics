---
name: univariate-cox-survival-analysis
description: Use when a bulk transcriptomic dataset has matched survival (event +
  time-to-event) information and you must test each gene-pair or marker-pair score
  individually for prognostic association before building a directed gene-gene network.
when_to_use_negative:
- The dataset lacks matched survival information or has too few samples/events — the
  paper notes TimiGP requires a large gene expression dataset with matched survival
  information to effectively identify prognostic gene pairs
- The analysis goal is differential expression or clustering rather than clinical
  outcome association; multivariate or machine-learning prognostic modeling is needed
  rather than per-pair univariate testing
- Input is therapy-response (binary responder/non-responder) labels only — use a logistic/response
  module (e.g., TimiFisher) instead of Cox regression
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: TimiGP
  role: R package providing TimiCOX (univariate Cox regression per marker pair) plus
    TimiCheckEvent, TimiPrePropress, and TimiGenePair that prepare its inputs
  repo: https://github.com/CSkylarL/TimiGP
- name: R
  role: Runtime environment; TimiGP installed via devtools::install_github
  repo: https://github.com/r-lib/devtools
- name: survival
  role: Underlying R survival-analysis machinery for fitting the Cox models
provenance:
  source_task_ids:
  - task_002
  - task_007
  - task_008
  - task_003
  source_papers:
  - doi: 10.1016/j.xcrm.2023.101121
    title: 'TimiGP: Inferring cell-cell interactions and prognostic associations in
      the tumor immune microenvironment through gene pairs'
  source_content_hash: sha256:372840a939fd9426a0b668ab28fc898b9bc674ccadb3658068b3ee2ab81b0692
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/CSkylarL/TimiGP
  tools:
  - R
  - TimiCOX
  - TimiFS
  - TimiGP
  - TimiPermFDR
  - survival
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# univariate-cox-survival-analysis

## Summary

Fit univariate Cox proportional-hazards regression on each gene/marker pair score against survival data (TimiCOX in the TimiGP framework) with Benjamini-Hochberg p-value adjustment, then orient each significant pair toward favorable prognosis by reversing pairs with HR>1.

## When to use

Use when a bulk transcriptomic dataset has matched survival (event + time-to-event) information and you must test each gene-pair or marker-pair score individually for prognostic association before building a directed gene-gene network.

## When NOT to use

- The dataset lacks matched survival information or has too few samples/events — the paper notes TimiGP requires a large gene expression dataset with matched survival information to effectively identify prognostic gene pairs
- The analysis goal is differential expression or clustering rather than clinical outcome association; multivariate or machine-learning prognostic modeling is needed rather than per-pair univariate testing
- Input is therapy-response (binary responder/non-responder) labels only — use a logistic/response module (e.g., TimiFisher) instead of Cox regression

## Inputs

- Marker Pair Score (MPS) matrix from TimiGenePair (binary pairwise expression comparisons of markers)
- Filtered clinical survival information (event and time-to-event) from TimiCheckEvent
- Preprocessed marker expression matrix (log-transformed, gene-wise median normalized via TimiPrePropress)

## Outputs

- Table of Cox regression results per marker pair (HR, p-value, BH-adjusted Q-value)
- Set of prognostic IMGP pairs with QV<0.05, each oriented so favorable-prognosis pairs have HR<=1 (HR>1 pairs reversed)
- Selected marker pairs usable as query input to TimiEnrich / TimiGeneNetwork

## How to apply

First pass the clinical information through TimiCheckEvent to filter low-quality event and time-to-event records, and preprocess marker expression with TimiPrePropress (log=TRUE, GMNorm=TRUE for gene-wise median normalization). Generate the binary Marker Pair Score (MPS) matrix with TimiGenePair (1 when gene A expression exceeds gene B, 0 otherwise). Run TimiCOX, which performs univariate Cox regression fitting each marker pair as a single variable, with p.adj='BH'; retain pairs with QV<0.05. Orient every retained pair toward favorable prognosis by reversing pairs with HR>1 so that A_B=1 consistently denotes a favorable-prognosis association. The rationale is that these directed prognostic IMGP selections feed the downstream gene-gene network, enrichment-based cell-cell interaction inference (TimiEnrich + TimiPermFDR), and favorability scoring (TimiFS).

## Related tools

- **TimiGP** (R package providing TimiCOX (univariate Cox regression per marker pair) plus TimiCheckEvent, TimiPrePropress, and TimiGenePair that prepare its inputs) — https://github.com/CSkylarL/TimiGP
- **R** (Runtime environment; TimiGP installed via devtools::install_github) — https://github.com/r-lib/devtools
- **survival** (Underlying R survival-analysis machinery for fitting the Cox models)

## Examples

```
res <- TimiCOX(mps = mps, info = SKCM06info, p.adj = "BH")
```

## Evaluation signals

- All retained marker pairs have BH-adjusted QV<0.05
- Every kept pair is oriented toward favorable prognosis: no retained pair has HR>1 (pairs with HR>1 were reversed)
- The Cox results table contains one fit (HR, p-value, Q-value) for every marker pair tested, matching the MPS matrix rows
- Downstream cell-cell interaction results are consistent, e.g., on TCGA_SKCM06 with Bindea2013 markers the pipeline yields 54/462 significant cell pairs with cytotoxic->tumor top-ranked and tumor cells with the highest unfavorable TimiFS score

## Limitations

- To effectively identify prognostic gene pairs, TimiGP requires a large gene expression dataset with matched survival information; statistical power depends on sample size, number of events, and survival-data quality
- Univariate fits each marker pair independently, so immune co-infiltration must be handled downstream (this is the bias TimiGP's network step is designed to resolve)
- Because downstream interaction calls are enrichment-based, cell types with fewer marker genes are less likely to be identified as having significant interactions

## Evidence
