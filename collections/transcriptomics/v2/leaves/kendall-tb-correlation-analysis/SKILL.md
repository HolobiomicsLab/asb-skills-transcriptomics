---
name: kendall-tb-correlation-analysis
description: 'Use when validating simulated scRNA-seq/scATAC-seq ground truth: you
  need to confirm that modelled activator/repressor gene–peak pairs show stronger
  within-group correlations than pairs with no simulated regulatory effect.'
when_to_use_negative:
- Validating bulk (non single-cell) simulated gene-regulator relationships — the paper
  specifies Pearson's correlation for bulk data and Kendall's Tb only for single-cell
  data
- Measuring correlation between features that both lack simulated regulatory effects
  as a primary result — these serve only as the null baseline against the modelled
  interaction set
- Inferring new regulatory interactions from real (non-simulated) data — this skill
  evaluates ground-truth recovery in simulated data, not de novo GRN inference
tools:
- name: MOSim
  role: R package whose sc_mosim function generates the simulated scRNA-seq and scATAC-seq
    data with modelled regulatory relationships that the Kendall Tb analysis validates
  repo: https://github.com/ConesaLab/MOSim
- name: R
  role: Implementation environment in which the aggregation, log(x+1) transformation,
    and Kendall Tb correlation computation are performed
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bib/bbaf110
    title: 'MOSim: bulk and single-cell multilayer regulatory network simulator'
  source_content_hash: sha256:7dbd9120c2d521eff46c098f6567c7024a6d8bd50e33f1e66b15a2a4701cceee
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
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# kendall-tb-correlation-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Validates simulated single-cell gene–peak regulatory relationships by computing Kendall's Tb correlations between gene and peak profiles within each experimental group, comparing modelled regulator interactions against a null of 20,000 random no-effect feature pairs (threshold: |Tb| > 0.7).

## When to use

Use when validating simulated scRNA-seq/scATAC-seq ground truth: you need to confirm that modelled activator/repressor gene–peak pairs show stronger within-group correlations than pairs with no simulated regulatory effect.

## When NOT to use

- Validating bulk (non single-cell) simulated gene-regulator relationships — the paper specifies Pearson's correlation for bulk data and Kendall's Tb only for single-cell data
- Measuring correlation between features that both lack simulated regulatory effects as a primary result — these serve only as the null baseline against the modelled interaction set
- Inferring new regulatory interactions from real (non-simulated) data — this skill evaluates ground-truth recovery in simulated data, not de novo GRN inference

## Inputs

- Simulated single-cell count matrices for scRNA-seq and scATAC-seq (e.g. produced by MOSim's sc_mosim, as in task_003 outputs)
- List of modelled gene-regulator pairs with effect type (activator / repressor / no effect)
- Experimental group / cell-type labels per cell

## Outputs

- Per-group Kendall Tb correlation table for modelled and random pairs (kendall_validation_report.csv)
- Percentage of activator/repressor interactions with |Kendall Tb| > 0.7 (target ~79.5%)
- Mean/centre of absolute correlation for the no-effect random pair set (target ~0.32)

## How to apply

Load the simulated scRNA-seq and scATAC-seq count matrices and aggregate the simulated count matrix to obtain the average count per cell type. Apply a log(x+1) transformation to the aggregated data, as specified in the MOSim methods. Compute Kendall's Tb correlation between gene and peak profiles within each experimental group for all modelled gene-regulator pairs (activators and repressors), using Kendall's Tb specifically because the paper designates it for single-cell data while Pearson's correlation is reserved for bulk data. Establish a null baseline by computing the same correlations for 20,000 random feature pairs with no simulated regulatory effects. Accept the simulation output if approximately 79.5% of modelled activator/repressor interactions exceed absolute Kendall Tb of 0.7, while the no-effect set centres around 0.32 absolute correlation.

## Related tools

- **MOSim** (R package whose sc_mosim function generates the simulated scRNA-seq and scATAC-seq data with modelled regulatory relationships that the Kendall Tb analysis validates) — https://github.com/ConesaLab/MOSim
- **R** (Implementation environment in which the aggregation, log(x+1) transformation, and Kendall Tb correlation computation are performed)

## Examples

```
Rscript -e 'agg <- aggregate(counts, by=list(celltype), FUN=mean); agg_log <- log(agg[,-1]+1); cors <- cor(t(agg_log), method="kendall")'
```

## Evaluation signals

- ~79.5% of modelled activator/repressor gene-peak interactions have absolute Kendall Tb correlation > 0.7
- The 20,000 random no-effect feature pairs centre around an absolute correlation of ~0.32, clearly separated from the modelled interaction distribution
- Correlations are computed within each experimental group (not pooled across groups) using the log(x+1)-transformed per-cell-type aggregated counts
- Per-group correlation values and both threshold checks are written to kendall_validation_report.csv

## Limitations

- Correlation strength depends on the underlying simulation quality; in MOSim, single-cell simulation is restricted to scRNA-seq and scATAC-seq only
- The 0.7 and ~0.32 values are simulation-specific benchmarks from the MOSim paper; other simulation settings may yield different expected distributions
- Aggregating to average count per cell type discards single-cell heterogeneity — this validation only checks cell-type-level regulatory signal

## Evidence

- [methods] Aggregation and log transform: [evidence span withheld — anchor status: not_found]
