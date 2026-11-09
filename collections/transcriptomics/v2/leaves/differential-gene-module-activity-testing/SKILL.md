---
name: differential-gene-module-activity-testing
description: Use when single-cell gene module activities (GMVs) have been inferred
  per cell in a VEGA latent space and you need statistically testable, per-module
  activity differences between conditions (e.g., Trametinib vs. DMSO).
when_to_use_negative:
- The latent space has not been wired to user-provided gene modules (GMVs) — differential
  testing requires the interpretable VEGA decoder, not a generic VAE embedding
- You need differential expression at the individual gene level rather than pathway/module
  level
- The model's generative capacity limits matter more than interpretability — VEGA's
  sparse single-layer decoder sacrifices predictive performance for interpretability
edam_operation: http://edamontology.org/operation_3566
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: VEGA
  role: Provides the Bayesian differential pathway activity testing procedure applied
    to per-cell GMV activities inferred in the gene-module-wired latent space
  repo: https://github.com/LucasESBS/vega
- name: scvi-tools
  role: Backbone implementation (vega.VegaSCVI) whose differential expression testing
    procedure inspired VEGA's module activity test
- name: scanpy
  role: Wraps the single-cell AnnData dataset and recommended preprocessing before
    VEGA training and testing
  repo: https://scanpy.readthedocs.io/en/stable/
provenance:
  source_task_ids:
  - task_003
  - task_006
  source_papers:
  - doi: 10.1038/s41467-021-26017-0
    title: VEGA is an interpretable generative model for inferring biological network
      activity in single-cell transcriptomics
  source_content_hash: sha256:1663315eecdb0a30e7cc8b81bb445b880819eff58201911c52f7a94e2a5b4322
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LucasESBS/vega
  tools:
  - MSigDB
  - VEGA
  - scanpy
  - scvi-tools
  edam_operation: http://edamontology.org/operation_3566
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# differential-gene-module-activity-testing

## Summary

Run VEGA's Bayesian differential pathway activity testing procedure — inspired by scvi-tools differential expression — to identify gene modules (GMVs) that are significantly activated or deactivated between conditions, such as drug treatment vs. DMSO control.

## When to use

Use when single-cell gene module activities (GMVs) have been inferred per cell in a VEGA latent space and you need statistically testable, per-module activity differences between conditions (e.g., Trametinib vs. DMSO).

## When NOT to use

- The latent space has not been wired to user-provided gene modules (GMVs) — differential testing requires the interpretable VEGA decoder, not a generic VAE embedding
- You need differential expression at the individual gene level rather than pathway/module level
- The model's generative capacity limits matter more than interpretability — VEGA's sparse single-layer decoder sacrifices predictive performance for interpretability

## Inputs

- Trained VEGA model(s) (one per condition, decoder mask from MSigDB hallmark GMT gene sets)
- Per-cell gene module activity (GMV) scores for each condition embedded in the interpretable latent space
- Condition labels separating treatment cells from control cells (e.g., drug vs. DMSO)

## Outputs

- Per-gene-module Bayesian differential activity statistics (log-evidence / log_e(BF)) between conditions
- Ranked list of significantly activated/deactivated gene modules per treatment-control comparison

## How to apply

First train one VEGA model per condition (e.g., per drug treatment plus one for the DMSO control), with the sparse linear decoder wired from MSigDB hallmark GMT files, and infer single-cell GMV activities in each trained model. Then apply VEGA's Bayesian testing procedure (inspired by scvi-tools differential expression) to each gene module between the treatment and control cells, yielding log-evidence (log BF) statistics per module. Interpret significant modules mechanistically: e.g., decreased G2M checkpoint GMV under Trametinib agrees with the expected mechanism of action of a MEK inhibitor, and Trametinib-specific interferon-alpha/gamma responses should surface. Decision point: treat average |log_e(BF)| magnitude as an effect-size signal — the paper reports a larger average |log_e(BF)| for Dabrafenib-treated BRAF-mutant melanoma lines (0.763) versus other lines (0.668), so compare evidence magnitudes across cell-line subgroups rather than relying on a single cutoff.

## Related tools

- **VEGA** (Provides the Bayesian differential pathway activity testing procedure applied to per-cell GMV activities inferred in the gene-module-wired latent space) — https://github.com/LucasESBS/vega
- **scvi-tools** (Backbone implementation (vega.VegaSCVI) whose differential expression testing procedure inspired VEGA's module activity test)
- **scanpy** (Wraps the single-cell AnnData dataset and recommended preprocessing before VEGA training and testing) — https://scanpy.readthedocs.io/en/stable/

## Examples

```
import vega; model = vega.VEGA(my_adata, gmt_file='msigdb_hallmark.gmt'); model.train(); results = model.differential_activity(group1='Trametinib', group2='DMSO')
```

## Evaluation signals

- Reproduced finding: decreased G2M checkpoint GMV activity under Trametinib, consistent with the expected mechanism of a MEK inhibitor
- Trametinib-specific interferon-alpha/gamma response modules appear among top differentially active GMVs
- Average |log_e(BF)| for Dabrafenib-treated BRAF-mutant melanoma lines (0.763) exceeds that of other lines (0.668)
- Significant modules are reported with Bayesian evidence (log BF) values, not just raw GMV means

## Limitations

- Prior biological knowledge from MSigDB can be incomplete or not context-specific, so unannotated genes important to a module are excluded by the hard-coded decoder mask
- The test operates on GMVs from a model with limited generative capacity; the sparse single-layer decoder limits generalizability and robustness
- Hard-coded decoder connections leave no room for correcting prior knowledge about gene modules when context requires it

## Evidence

- [methods] Bayesian differential pathway activity testing: "VEGA provides a Bayesian testing procedure to find significantly activated gene modules in your dataset"
- [results] MIX-Seq validation of drug-response GMVs: [evidence span withheld — anchor status: not_found]
- [methods] Preprocessing recommendation: "Note: We recommend to preprocess the single-cell dataset before passing it to VEGA."
- [readme] GMT module definition: [evidence span withheld — anchor status: external_doc]
- [abstract] Architecture interpretability rationale: "whose decoder wiring mirrors user-provided gene modules, providing direct interpretability to the latent variables"
