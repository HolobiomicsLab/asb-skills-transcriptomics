---
name: null-baseline-random-pair-comparison
description: Use when validating simulated or inferred gene–regulator relationships
  against ground truth, to establish whether modelled activator/repressor interactions
  show stronger correlations than random feature pairs with no simulated regulatory
  effects.
when_to_use_negative:
- Bulk multi-omics data validation — use Pearson's correlation instead of Kendall's
  Tb, as MOSim specifies different statistics per data modality
- When no simulated ground-truth regulator set exists — the null comparison requires
  known activator/repressor pairs and true no-effect pairs
- Omics layers outside MOSim's scope (proteomics, metabolomics, spatially resolved
  data) that the simulator cannot generate
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: MOSim
  role: R package that simulates the scRNA-seq and scATAC-seq data and provides the
    ground-truth regulatory interactions used for the Kendall Tb validation against
    random no-effect pairs
  repo: github.com/ConesaLab/MOSim
- name: R
  role: Language in which MOSim is implemented and in which aggregation, log(x+1)
    transformation, and Kendall Tb correlation computation are performed
  repo: github.com/ConesaLab/MOSim
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bib/bbaf110
    title: 'MOSim: bulk and single-cell multilayer regulatory network simulator'
  source_content_hash: sha256:f317a58d0941ed9b745d322d9887cdb94988aa1eeac7a912312306b95a047f79
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
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 4
---

# Null-baseline random pair comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Validates simulated regulatory interactions by comparing their absolute correlations against a null baseline of ~20,000 random feature pairs with no simulated regulatory effects (e.g. Kendall's Tb for single-cell gene–peak pairs, Pearson for bulk).

## When to use

Use when validating simulated or inferred gene–regulator relationships against ground truth, to establish whether modelled activator/repressor interactions show stronger correlations than random feature pairs with no simulated regulatory effects.

## When NOT to use

- Bulk multi-omics data validation — use Pearson's correlation instead of Kendall's Tb, as MOSim specifies different statistics per data modality
- When no simulated ground-truth regulator set exists — the null comparison requires known activator/repressor pairs and true no-effect pairs
- Omics layers outside MOSim's scope (proteomics, metabolomics, spatially resolved data) that the simulator cannot generate

## Inputs

- Simulated single-cell RNA-seq count matrix (scRNA-seq) from MOSim sc_mosim
- Simulated single-cell ATAC-seq peak count matrix (scATAC-seq) from MOSim sc_mosim
- Set of modelled activator/repressor gene–regulator interactions (ground truth from simulation)
- Set of 'no effect' / random feature pairs without simulated regulatory effects

## Outputs

- Per-group correlation table for modelled and random feature pairs
- Percentage of activator/repressor interactions with absolute Kendall Tb > 0.7 (e.g. 79.5%)
- Mean/centre absolute correlation of the no-effect pair set (~0.32)
- Kendall validation report (e.g. kendall_validation_report.csv)

## How to apply

From the simulated single-cell count matrices (e.g. from sc_mosim), aggregate the count matrix to obtain the average count per cell type, then apply a log(x+1) transformation as specified in the MOSim methods. Compute Kendall's Tb correlation between gene and peak profiles within each experimental group for all modelled gene-regulator pairs, and separately compute the same correlations for 20,000 random feature pairs with no simulated regulatory effects to establish the null baseline. Judge success by the separation: approximately 79.5% of modelled activator/repressor interactions should exceed absolute Kendall Tb correlation 0.7, while the no-effect pairs should centre around an absolute correlation of ~0.32. For bulk data, Pearson's correlation is used instead of Kendall's Tb.

## Related tools

- **MOSim** (R package that simulates the scRNA-seq and scATAC-seq data and provides the ground-truth regulatory interactions used for the Kendall Tb validation against random no-effect pairs) — github.com/ConesaLab/MOSim
- **R** (Language in which MOSim is implemented and in which aggregation, log(x+1) transformation, and Kendall Tb correlation computation are performed) — github.com/ConesaLab/MOSim

## Examples

```
BiocManager::install("MOSim"); # then: aggregate scRNA-seq/scATAC-seq counts per cell type, log(x+1) transform, and compute Kendall Tb for regulatory vs. 20000 random no-effect pairs
```

## Evaluation signals

- Approximately 79.5% of modelled activator/repressor gene–peak interactions exceed absolute Kendall Tb correlation 0.7
- No-effect random feature pairs (20,000 sampled) centre around an absolute correlation of ~0.32
- Aggregated single-cell counts are log(x+1)-transformed before correlation, matching the MOSim methods
- Correlations are computed within each experimental group, not pooled across groups
- The same correlation statistic (Kendall's Tb for single-cell, Pearson for bulk) is applied to both the regulatory and null pair sets

## Limitations

- MOSim single-cell simulation is restricted to scRNA-seq and scATAC-seq, so this validation only covers gene–peak (and TF) relationships for those layers
- The validation depends on the simulator's ground truth; it cannot be applied to data without known active vs. inactive regulatory pairs

## Evidence

- [methods] a log transformation (log(x+1)) [42] was applied to the data: [evidence span withheld — anchor status: not_found]
