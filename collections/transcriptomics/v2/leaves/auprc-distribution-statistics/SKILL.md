---
name: auprc-distribution-statistics
description: Use when benchmarking GRN reconstruction against a gold-standard interaction
  network and you need AUPRC distributions under partial-groundtruth train/test splits
  (e.g. 0.5%-10% gold-standard fractions) rather than a single full-groundtruth AUPRC.
when_to_use_negative:
- You only need a single AUPR/AUROC point estimate against a fully available groundtruth
  — use the au_pr_roc binary instead of the train/test-split ensemble evaluation
- No gold-standard interaction network exists for your organism — eval_mcpnet's train/test
  splitting requires a groundtruth, which is why it is 'suitable when full groundtruth
  is available, hence synthetic expression data'
- Input is raw expression data not yet converted into candidate method score matrices
  (MI/MCP scores) — compute those first with the mi/mcp pipeline
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: MCPNet
  role: Provides the eval_mcpnet executable that divides the groundtruth into training/testing
    sets and evaluates ensemble AUPR distributions over user-specified iterations,
    plus the mi/mcp/transform binaries that produce the candidate score matrices being
    ensembled
  repo: https://github.com/AluruLab/MCPNet
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad373
    title: 'MCPNet: a parallel maximum capacity-based genome-scale gene network construction
      framework'
  source_content_hash: sha256:bc67ba7bb6b3a3759ad08d2423f14b4e3bdedd2a56da349a6c315d3c3d47b7b3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/AluruLab/MCPNet
  tools:
  - MCPNet
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# auprc-distribution-statistics

## Summary

Computes distributions of AUPRC (area under the precision-recall curve) for gene regulatory network reconstruction methods when only a partial groundtruth is available, by splitting the gold-standard network into train/test sets across repeated iterations using MCPNet's eval_mcpnet executable.

## When to use

Use when benchmarking GRN reconstruction against a gold-standard interaction network and you need AUPRC distributions under partial-groundtruth train/test splits (e.g. 0.5%-10% gold-standard fractions) rather than a single full-groundtruth AUPRC.

## When NOT to use

- You only need a single AUPR/AUROC point estimate against a fully available groundtruth — use the au_pr_roc binary instead of the train/test-split ensemble evaluation
- No gold-standard interaction network exists for your organism — eval_mcpnet's train/test splitting requires a groundtruth, which is why it is 'suitable when full groundtruth is available, hence synthetic expression data'
- Input is raw expression data not yet converted into candidate method score matrices (MI/MCP scores) — compute those first with the mi/mcp pipeline

## Inputs

- Gene expression matrix in HDF5 format (e.g. simulated/noise-free yeast expression, gnw2000-style data)
- Gold-standard groundtruth interaction network (CSV, e.g. gnw2000_truenet.csv or yeast gold-standard)
- Candidate method score matrices computed by MCPNet binaries (mi, mcp) with optional Stouffer/CLR transforms
- User-specified number of ensemble iterations and list of gold-standard fractions (0.5%-10%)

## Outputs

- Per-iteration test-AUPRC values for the GCN and GRN ensembles at each gold-standard fraction
- Mean and standard deviation of AUPRC distributions per fraction and ensemble type
- Summary table (e.g. yeast_ensemble_auprc_distributions.csv)

## How to apply

Run eval_mcpnet with the gene expression matrix (-i, HDF5), the groundtruth network (-x), the same -m method codes and --mi-method parameters used for the candidate score matrices (via mi/mcp plus Stouffer/CLR transforms from the transform utility), and the desired number of ensemble iterations (e.g. 100). At each gold-standard fraction (0.5%, 1%, 2%, 5%, 10%), the groundtruth is divided into training and testing sets; the ensemble combination coefficients are optimized on the training subset and the held-out testing subset is scored with the combined ensemble ranking, yielding one test-AUPRC per iteration. Aggregate the per-iteration test-AUPRC values into mean and standard deviation distributions for each ensemble type (GCN and GRN) and fraction, then verify means fall within expected ranges (yeast: GCN 0.4135-0.4201, GRN 0.4979-0.5070) and compare against the full-groundtruth AUPRC (0.4198) to assess robustness. Write the aggregated summary table (e.g. yeast_ensemble_auprc_distributions.csv) for downstream plotting.

## Related tools

- **MCPNet** (Provides the eval_mcpnet executable that divides the groundtruth into training/testing sets and evaluates ensemble AUPR distributions over user-specified iterations, plus the mi/mcp/transform binaries that produce the candidate score matrices being ensembled) — https://github.com/AluruLab/MCPNet

## Examples

```
{MCPNet_build_dir}/bin/au_pr_roc -i gnw2000.mcp4.h5 -x {MCPNet_source_dir}/data/gnw2000_truenet.csv
```

## Evaluation signals

- Reproduced mean test-AUPRCs for the yeast GCN ensemble fall within 0.4135-0.4201 across the 0.5%-10% fractions
- Reproduced mean test-AUPRCs for the yeast GRN ensemble fall within 0.4979-0.5070 across the same fractions
- Means are close to the full-groundtruth AUPRC of 0.4198, confirming partial-groundtruth robustness
- Each fraction contains a distribution over the full set of user-specified iterations (100 in the reproduced experiment), not a single value
- Summary CSV (yeast_ensemble_auprc_distributions.csv) is produced with mean and standard deviation columns per fraction and ensemble type

## Limitations

- eval_mcpnet requires a full groundtruth to split, making it appropriate mainly for synthetic/simulated expression data rather than purely experimental settings
- AUPRC distributions depend on the random train/test splits, so enough iterations must be specified to capture the distribution reliably
- Results depend on the method codes (-m) and MI-method parameters passed to the binaries; the same parameters used to compute candidate score matrices must be supplied to eval_mcpnet

## Evidence

- [readme] eval_mcpnet:  this executable is used for evaluating the performance of the ensemble method when only partial truth is provided.: "eval_mcpnet:  this executable is used for evaluating the performance of the ensemble method when only partial truth is provided."
- [readme] The groundtruth is divided into training and testing sets and the AUPR is evaluated for as many iterations as the user specifies.: "The groundtruth is divided into training and testing sets and the AUPR is evaluated for as many iterations as the user specifies."
- [abstract] we demonstrate that MCPNet produces better quality networks as measured by AUPRC, is significantly faster than all other gene network reconstruction software: "we demonstrate that MCPNet produces better quality networks as measured by AUPRC, is significantly faster than all other gene network reconstruction software"
