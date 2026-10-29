---
name: surface-marker-enrichment-testing
description: Use when single-cell RNA-Seq data has both per-cell signalling entropy
  scores (from SCENT) and known surface-marker annotations (e.g. CD34/CD96), and you
  must verify that top-entropy cells correspond to the marker-defined CSC population.
when_to_use_negative:
- The dataset is bulk RNA-Seq with no per-cell resolution — use SCENT's bulk estimation
  to compare SR between sample groups instead of per-cell marker enrichment
- No validated surface markers exist for the presumed stem/drug-resistant population,
  so no independent enrichment test can be constructed
- The goal is pseudo-time ordering of cells rather than validating entropy against
  a marker-defined subpopulation
edam_operation: http://edamontology.org/operation_3501
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: SCENT
  role: Computes per-cell signalling entropy used to rank cells by differentiation
    potency prior to marker enrichment testing
  repo: https://github.com/aet21/SCENT
- name: R
  role: Environment in which SCENT is run and the enrichment tests and output tables
    are produced
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/ncomms15599
    title: Single-cell entropy for accurate estimation of differentiation potency
      from a cell’s transcriptome
  source_content_hash: sha256:70346770ac682eacf03c181f67f868460649353b30109bab7c102939af0f5cdc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/aet21/SCENT
  tools:
  - BiocStyle
  - CCAT
  - R
  - SCENT
  edam_operation: http://edamontology.org/operation_3501
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# surface-marker-enrichment-testing

## Summary

Validate that SCENT-derived signalling entropy rankings capture known stem/drug-resistant phenotypes by testing whether canonical surface markers (e.g. CD34, CD96 in AML) are enriched among the highest-entropy cells. This links an unsupervised potency estimate to independent marker-based cell identity.

## When to use

Use when single-cell RNA-Seq data has both per-cell signalling entropy scores (from SCENT) and known surface-marker annotations (e.g. CD34/CD96), and you must verify that top-entropy cells correspond to the marker-defined CSC population.

## When NOT to use

- The dataset is bulk RNA-Seq with no per-cell resolution — use SCENT's bulk estimation to compare SR between sample groups instead of per-cell marker enrichment
- No validated surface markers exist for the presumed stem/drug-resistant population, so no independent enrichment test can be constructed
- The goal is pseudo-time ordering of cells rather than validating entropy against a marker-defined subpopulation

## Inputs

- Per-cell signalling entropy (SR) scores from SCENT applied to a single-cell RNA-Seq matrix
- Single-cell RNA-Seq expression data with surface-marker information (e.g. AML scRNA-Seq annotated for CD34 and CD96)
- A definition of the 'highest-entropy' cell set (top fraction of SR-ranked cells)

## Outputs

- Enrichment P-values for each surface marker among the highest-entropy cells (e.g. CD34 P=0.008, CD96 P=0.032)
- Entropy-ranked cell list with marker-positive/negative labels
- Comparison table entries (e.g. rows in aml_ctc_scent_reproduction.csv)

## How to apply

Run SCENT on the single-cell expression matrix to compute a signalling entropy (SR) value per cell, then rank cells by SR. Take the highest-entropy cells and test for enrichment of the surface-marker-positive cells among them (in the AML reproduction this targets CD34 enrichment P=0.008 and CD96 enrichment P=0.032). The rationale is that signalling entropy independently orders single cells by differentiation potency without needing marker or timepoint information, so agreement between high entropy and marker positivity is an independent correctness check of the potency ranking. Report the enrichment P-values alongside the SR distributions so the marker-based and entropy-based orderings can be compared directly.

## Related tools

- **SCENT** (Computes per-cell signalling entropy used to rank cells by differentiation potency prior to marker enrichment testing) — https://github.com/aet21/SCENT
- **R** (Environment in which SCENT is run and the enrichment tests and output tables are produced)

## Examples

```
devtools::install_github("aet21/SCENT"); SR <- SCENT::Entropy(expr.sc, gene.info, sig.network)$SR; ranks <- order(SR, decreasing=TRUE); table(marker[ranks[1:n_top]] )
```

## Evaluation signals

- CD34 enrichment among the highest-entropy AML cells reproduces the reported P=0.008
- CD96 enrichment among the highest-entropy AML cells reproduces the reported P=0.032
- Every cell in the SCENT output has a finite SR value so the entropy ranking is complete before enrichment testing
- Relapsed AML bulk samples show higher SR than matched diagnostic samples (paired Wilcoxon P=0.004), consistent with entropy marking the drug-resistant compartment
- Reproduced P-values are compiled into the output table aml_ctc_scent_reproduction.csv without missing comparison rows

## Limitations

- Agreement between entropy ranking and markers is correlative; the precise relation between signalling entropy and other entropy definitions is non-trivial and not established
- High dropout in scRNA-Seq can preclude reliance on a single marker gene — which is the motivation for using entropy, but also means marker calls themselves may be noisy
- Enrichment depends on the choice of 'highest-entropy' threshold; the article does not prescribe a universal cutoff, so sensitivity to that choice should be checked

## Evidence

- [abstract] Signalling entropy identifies known cell subpopulations of varying potency and drug resistant cancer stem-cell phenotypes, including those derived from circulating tumour cells: "Signalling entropy identifies known cell subpopulations of varying potency and drug resistant cancer stem-cell phenotypes, including those derived from circulating tumour cells"
- [intro] The main purpose of the `SCENT` package is to provide a means of estimating the differentiation potency of single cells without the need to assume prior biological knowledge: "The main purpose of the `SCENT` package is to provide a means of estimating the differentiation potency of single cells without the need to assume prior biological knowledge"
- [other] relapsed AML bulk samples exhibit higher entropy than matched diagnostic samples (paired Wilcoxon test P=0.004): [evidence span withheld — anchor status: not_found]
- [readme] This may be particularly important in scenarios where a high dropout rate may preclude the use of a marker gene: "This may be particularly important in scenarios where a high dropout rate may preclude the use of a marker gene"
