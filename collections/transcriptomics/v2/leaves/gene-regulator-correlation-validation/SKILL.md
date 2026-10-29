---
name: gene-regulator-correlation-validation
description: Use when you have simulated or inferred multi-omics data with a ground-truth
  list of regulator–gene interactions and need to confirm that modelled regulatory
  effects produce detectable correlations distinct from a no-effect background.
when_to_use_negative:
- When the omics data contain no regulatory interaction model or ground truth — correlations
  would have no class labels to compare against the 20,000 random-pair background.
- 'When spatial resolution is required: MOSim is not prepared to simulate GRNs with
  spatial resolution, so this validation cannot be applied to spatial multi-omics
  data.'
- For proteomics or metabolomics layers, which MOSim does not simulate and therefore
  cannot be validated with this gene–regulator correlation approach.
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: MOSim
  role: Generates the simulated multi-omics count data and ground-truth regulatory
    interactions that are validated via correlation analysis
  repo: github.com/ConesaLab/MOSim
- name: R
  role: Implementation language for log(x+1) transformation, Pearson/Kendall Tb correlation
    computation, and random-pair background generation
- name: SPARSim
  role: Underlying functionality for sc_mosim single-cell count simulation feeding
    the Kendall Tb validation
provenance:
  source_task_ids:
  - task_005
  - task_004
  - task_002
  source_papers:
  - doi: 10.1093/bib/bbaf110
    title: 'MOSim: bulk and single-cell multilayer regulatory network simulator'
  source_content_hash: sha256:ae307d81efe9ad9ccdcc043e31f31c47bb21c1b51a37b07a0256060c16ff717b
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
  - SPARSim
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# gene-regulator-correlation-validation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Validate simulated multilayer gene regulatory network interactions by correlating regulator–gene expression profiles (Pearson's for bulk RNA-seq/ATAC-seq, Kendall's Tb for single-cell), comparing modelled activator/repressor pairs against 20,000 random no-effect feature pairs.

## When to use

Use when you have simulated or inferred multi-omics data with a ground-truth list of regulator–gene interactions and need to confirm that modelled regulatory effects produce detectable correlations distinct from a no-effect background.

## When NOT to use

- When the omics data contain no regulatory interaction model or ground truth — correlations would have no class labels to compare against the 20,000 random-pair background.
- When spatial resolution is required: MOSim is not prepared to simulate GRNs with spatial resolution, so this validation cannot be applied to spatial multi-omics data.
- For proteomics or metabolomics layers, which MOSim does not simulate and therefore cannot be validated with this gene–regulator correlation approach.

## Inputs

- Simulated bulk RNA-seq count matrix with regulator layers (ATAC-seq, miRNA-seq, ChIP-seq, Methyl-seq, TF) from mosim()
- Simulated scRNA-seq and scATAC-seq count matrices from sc_mosim()
- Ground-truth regulator–gene interaction list with interaction class (activator, repressor, no-effect)
- Experimental group/replicate metadata for per-group stratification

## Outputs

- Per-pair correlation table per experimental group and interaction class (e.g. regulator_correlation_validation.csv, kendall_validation_report.csv)
- Class-level summary: fraction of activator/repressor pairs above correlation threshold vs. no-effect random-pair background
- Validation verdict (pass/fail) against paper benchmarks (99.4% > 0.9 bulk; 79.5% > 0.7 single-cell)

## How to apply

Load the simulated count matrices (bulk RNA-seq with ATAC-seq/miRNA-seq/TF layers, or scRNA-seq with scATAC-seq) together with the ground-truth regulator–gene interaction list. For single-cell data, first aggregate the simulated count matrix to obtain the average count per cell type; apply a log(x+1) transformation to all count data as specified in the MOSim methods. Compute Pearson's correlation for bulk regulator–gene pairs per experimental group (optionally with lagged correlations up to a maximum lag of 2 to capture temporal delays), and Kendall's Tb correlation for single-cell pairs. Compute the same correlations for 20,000 random feature pairs with no simulated regulatory effects to establish the null baseline. Accept validation when modelled activator/repressor pairs exceed thresholds seen in the paper's examples: ~99.4% of bulk ATAC-seq interactions with |correlation| > 0.9 (0% of no-effect pairs), and ~79.5% of single-cell gene–peak interactions with absolute Kendall Tb > 0.7 while no-effect pairs centre around 0.32 absolute correlation.

## Related tools

- **MOSim** (Generates the simulated multi-omics count data and ground-truth regulatory interactions that are validated via correlation analysis) — github.com/ConesaLab/MOSim
- **R** (Implementation language for log(x+1) transformation, Pearson/Kendall Tb correlation computation, and random-pair background generation)
- **SPARSim** (Underlying functionality for sc_mosim single-cell count simulation feeding the Kendall Tb validation)

## Examples

```
Rscript validate_regulator_correlations.R --sim mosim_output.rds --interactions ground_truth.rds --n-random 20000 --max-lag 2 --out regulator_correlation_validation.csv
```

## Evaluation signals

- Approximately 99.4% of modelled bulk activator/repressor interactions show |Pearson correlation| > 0.9, with 0% of no-effect random pairs above that threshold (ATAC-seq example)
- Approximately 79.5% of single-cell gene–peak interactions with modelled effects show absolute Kendall Tb > 0.7, while no-effect interactions centre around 0.32 absolute correlation
- Correlations for modelled pairs clearly separate from the 20,000 random no-effect feature-pair background in every experimental group
- Per-group correlation table contains non-missing correlations for all activator, repressor, and no-effect classes after log(x+1) transformation

## Limitations

- Single-cell validation is restricted to scRNA-seq and scATAC-seq, the only two techniques that can be simultaneously performed on the same cell; other single-cell omics cannot be validated this way.
- Bulk validation may require lagged correlations (up to lag 2) to capture temporal delays between regulator activity and gene expression; unlagged correlations may miss true interactions.
- MOSim does not model proteomics/metabolomics effects, so regulatory influence of those layers cannot be validated with correlations.

## Evidence

- [methods] log(x+1) transformation applied before correlation: [evidence span withheld — anchor status: not_found]
- [readme] README citation and citation of SPARSim dependence for sc_mosim: [evidence span withheld — anchor status: external_doc]
