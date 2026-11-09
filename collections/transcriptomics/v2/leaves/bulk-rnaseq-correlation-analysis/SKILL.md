---
name: bulk-rnaseq-correlation-analysis
description: Use when you have simulated or ground-truth-annotated bulk RNA-seq and
  ATAC-seq count matrices plus a known regulator-gene interaction list and need to
  quantify whether modelled activator/repressor effects manifest as strong expression-accessibility
  correlations.
when_to_use_negative:
- 'Data are single-cell (scRNA-seq/scATAC-seq): the article specifies Kendall''s Tb
  correlation for single-cell regulator-gene validation, not Pearson''s on bulk counts.'
- You have no ground-truth or simulated regulator-gene interaction annotation, so
  activator/repressor/no-effect classes cannot be assigned and the random-pair comparison
  is meaningless.
- The goal is differential expression or differential accessibility testing rather
  than validating whether simulated regulatory interactions manifest as strong correlations.
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3173
tools:
- name: MOSim
  role: R package that generates the simulated bulk RNA-seq and ATAC-seq count data
    and the active regulator-gene interaction ground truth that this correlation validation
    procedure evaluates.
  repo: github.com/ConesaLab/MOSim
- name: R
  role: Language/environment in which MOSim runs and in which the log(x+1) transformation,
    Pearson correlations with lag up to 2, and random-pair null generation are computed.
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bib/bbaf110
    title: 'MOSim: bulk and single-cell multilayer regulatory network simulator'
  source_content_hash: sha256:3f7fe64c09da132c76dbb348280ab83f2a3cc8c00c5a84587643c5914cba364a
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/ConesaLab/MOSim
  tools:
  - MOSim
  - R
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 5
---

# bulk-rnaseq-correlation-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Validation procedure for simulated (or ground-truth-annotated) bulk regulatory networks: compute Pearson correlations between regulator accessibility/activity and gene expression per regulator-gene pair, including lagged correlations, and benchmark them against a null set of 20,000 random feature pairs with no simulated regulatory effect.

## When to use

Use when you have simulated or ground-truth-annotated bulk RNA-seq and ATAC-seq count matrices plus a known regulator-gene interaction list and need to quantify whether modelled activator/repressor effects manifest as strong expression-accessibility correlations.

## When NOT to use

- Data are single-cell (scRNA-seq/scATAC-seq): the article specifies Kendall's Tb correlation for single-cell regulator-gene validation, not Pearson's on bulk counts.
- You have no ground-truth or simulated regulator-gene interaction annotation, so activator/repressor/no-effect classes cannot be assigned and the random-pair comparison is meaningless.
- The goal is differential expression or differential accessibility testing rather than validating whether simulated regulatory interactions manifest as strong correlations.

## Inputs

- Simulated bulk RNA-seq count matrix (e.g. from MOSim mosim function)
- Simulated bulk ATAC-seq count matrix
- Ground-truth regulator-gene interaction list (activator/repressor/no-effect classes) from the simulation
- Experimental group / time-point design labels

## Outputs

- Per-pair correlation table written to regulator_correlation_validation.csv
- Class-level summary of the fraction of pairs with |correlation| > 0.9 for activator, repressor, and no-effect interactions
- Null background correlation distribution from 20,000 random feature pairs

## How to apply

Load the simulated bulk RNA-seq and ATAC-seq count matrices and the ground-truth regulator-gene interaction list, then apply a log(x+1) transformation to the counts as described in the MOSim methods. For each regulator-gene pair per experimental group, compute Pearson's correlation (the correlation used for bulk data, versus Kendall's Tb for single-cell), including lagged correlations up to a maximum lag of 2 to capture temporal delays between regulator activity and gene expression. Compute the same correlations for 20,000 random feature pairs with no simulated regulatory effects to establish the null background. Summarize per interaction class (activator, repressor, no-effect) the fraction of pairs with |correlation| > 0.9 and compare modelled pairs against the random-pair background; in the MOSim ATAC-seq example, 99.4% of modelled activator/repressor interactions exceeded 0.9 while 0% of no-effect pairs did. Write per-pair correlations and class-level summaries to regulator_correlation_validation.csv.

## Related tools

- **MOSim** (R package that generates the simulated bulk RNA-seq and ATAC-seq count data and the active regulator-gene interaction ground truth that this correlation validation procedure evaluates.) — github.com/ConesaLab/MOSim
- **R** (Language/environment in which MOSim runs and in which the log(x+1) transformation, Pearson correlations with lag up to 2, and random-pair null generation are computed.)

## Examples

```
BiocManager::install("MOSim"); # then per regulator-gene pair: cor(log(as.numeric(rna[g,])+1), lagged_atac, method="pearson") with lag up to 2
```

## Evaluation signals

- Class-level summary shows approximately 99.4% of modelled activator/repressor interactions with |correlation| > 0.9 in the ATAC-seq example.
- 0% of no-effect interactions exceed the |correlation| > 0.9 threshold, matching the random-pair background.
- Per-pair correlation file regulator_correlation_validation.csv exists and contains correlations for every ground-truth interaction plus the 20,000 random pairs.
- Lagged correlations up to lag 2 were computed per experimental group, not only contemporaneous Pearson correlations.
- Counts were log(x+1)-transformed before correlation, consistent with the MOSim methods.

## Limitations

- Pearson correlation is the bulk-specific validation statistic; the article uses Kendall's Tb for single-cell data, so results do not transfer directly to sc_mosim output.
- The null background is defined by 20,000 random feature pairs with no simulated regulatory effects; conclusions depend on the correctness of the simulated no-effect annotation.
- Correlation validation captures linear (and lag-shifted linear) association only and may miss non-monotonic regulatory effects.

## Evidence

- [methods] Pearson's correlation validation for bulk gene-regulator relationships: "Pearson’s correlation was computed for bulk data and Kendall’s Tb correlation for single-cell data [43]"
- [methods] Random feature pair null background: "These correlations were compared with 20 000 random feature pairs with no simulated regulatory effects"
- [methods] Log transformation of count data: [evidence span withheld — anchor status: not_found]
