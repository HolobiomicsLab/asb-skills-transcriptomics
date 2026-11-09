---
name: null-distribution-comparison-with-random-feature-pairs
description: Use when you have ground-truth regulator-gene interactions from simulated
  multi-omics data and need to verify that modelled activator/repressor pairs show
  stronger correlations than unmodelled pairs.
when_to_use_negative:
- Input is single-cell simulated data — the article prescribes Kendall's Tb correlation
  for single-cell rather than Pearson's correlation for bulk
- The omics layers are proteomics or metabolomics, which MOSim cannot simulate and
  for which the regulatory correlation validation was not designed
edam_operation: http://edamontology.org/operation_3463
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0091
tools:
- name: MOSim
  role: R package that generates the simulated bulk RNA-seq and ATAC-seq data plus
    the ground-truth active regulatory relationships being validated
  repo: https://github.com/ConesaLab/MOSim
- name: R
  role: Implementation environment for log(x+1) transformation, Pearson/lagged correlation
    computation, and random-pair null construction
  repo: https://github.com/ConesaLab/MOSim
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bib/bbaf110
    title: 'MOSim: bulk and single-cell multilayer regulatory network simulator'
  source_content_hash: sha256:56ca89c7ce9d08b7332faf9353a71f0064449fafeae62233a24245a850679ae5
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/ConesaLab/MOSim
  repo_url: https://github.com/ConesaLab/MOSim
  tools:
  - MOSim
  - R
  edam_operation: http://edamontology.org/operation_3463
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# null-distribution-comparison-with-random-feature-pairs

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Validates simulated or inferred regulator-gene regulatory interactions by comparing their expression-accessibility correlations against a null distribution built from random feature pairs with no modelled regulatory effect.

## When to use

Use when you have ground-truth regulator-gene interactions from simulated multi-omics data and need to verify that modelled activator/repressor pairs show stronger correlations than unmodelled pairs.

## When NOT to use

- Input is single-cell simulated data — the article prescribes Kendall's Tb correlation for single-cell rather than Pearson's correlation for bulk
- The omics layers are proteomics or metabolomics, which MOSim cannot simulate and for which the regulatory correlation validation was not designed

## Inputs

- Simulated bulk RNA-seq count matrix (from MOSim mosim output)
- Simulated ATAC-seq count matrix (from MOSim mosim output)
- Ground-truth regulator-gene interaction list with class labels (activator, repressor, no-effect)

## Outputs

- Per-pair correlation values for modelled and random feature pairs (regulator_correlation_validation.csv)
- Class-level summary of fraction of pairs with |correlation| > 0.9 per interaction class (activator, repressor, no-effect)

## How to apply

Load the simulated bulk RNA-seq and ATAC-seq count matrices together with the ground-truth regulator-gene interaction list, and apply a log(x+1) transformation to the counts as described in the MOSim methods. Compute Pearson's correlation for each regulator-gene pair per experimental group, including lagged correlations up to a maximum lag of 2 to capture temporal delays between regulator activity and gene expression (note: the article uses Kendall's Tb for single-cell data instead). Compute the same correlations for 20,000 random feature pairs with no simulated regulatory effects to establish the null background. Summarize, per interaction class (activator, repressor, no-effect), the fraction of pairs with |correlation| > 0.9, and compare modelled pairs against the random-pair background; in the MOSim ATAC-seq example, ~99.4% of modelled activator/repressor interactions exceeded 0.9 while 0% of no-effect interactions did.

## Related tools

- **MOSim** (R package that generates the simulated bulk RNA-seq and ATAC-seq data plus the ground-truth active regulatory relationships being validated) — https://github.com/ConesaLab/MOSim
- **R** (Implementation environment for log(x+1) transformation, Pearson/lagged correlation computation, and random-pair null construction) — https://github.com/ConesaLab/MOSim

## Examples

```
log_counts <- log(counts + 1); cors <- cor(t(log_counts[regulators, ]), t(log_counts[genes, ])); sum(abs(cors) > 0.9) / length(cors)
```

## Evaluation signals

- Approximately 99.4% of modelled activator/repressor interactions show |correlation| > 0.9
- 0% of no-effect (random) interactions exceed the |correlation| > 0.9 threshold
- Lagged correlations computed up to a maximum lag of 2 per experimental group
- Null background built from exactly 20,000 random feature pairs with no simulated regulatory effects
- Per-pair correlations and class-level summaries written to regulator_correlation_validation.csv

## Limitations

- Single-cell simulation in MOSim is restricted to scRNA-seq and scATAC-seq only, limiting the correlation-validation approach to those layers in single-cell mode
- The skill depends on ground-truth interaction labels and is therefore only applicable to simulated data with known regulatory effects, not to real experimental data without a null design

## Evidence

- [methods] a log transformation (log(x+1)) was applied to the data: [evidence span withheld — anchor status: not_found]
