---
name: aupr-auroc-network-evaluation
description: Use when a reconstructed GRN or GCN (e.g., MCP score matrix like gnw2000.mcp4.h5,
  or output from Pearson/MI/DPI) must be scored for network quality against a known
  ground-truth edge list (e.g., gnw2000_truenet.
when_to_use_negative:
- No ground-truth network exists for the dataset (e.g., real yeast RNA-seq data without
  a validated edge list) — AUPR/AUROC cannot be computed; au_pr_roc requires a ground
  truth matrix via -x
- Input is a raw gene expression matrix rather than an inferred interaction/score
  matrix — infer the network first (mi, mcp, pearson) before evaluation
- Evaluating the ensemble method with only partial truth is needed as a training/testing
  split — use eval_mcpnet instead of plain au_pr_roc
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: MCPNet
  role: au_pr_roc computes AUC for PR and ROC curves against a ground-truth matrix;
    mcpnet runs MI computation, MCP score calculation, and AUPR/AUROC evaluation in
    one pass when -x is supplied
  repo: https://github.com/AluruLab/MCPNet
- name: C++
  role: Implementation language of the MCPNet binaries (au_pr_roc, mcpnet) that perform
    the evaluation
  repo: https://github.com/AluruLab/MCPNet
provenance:
  source_task_ids:
  - task_002
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad373
    title: 'MCPNet: a parallel maximum capacity-based genome-scale gene network construction
      framework'
  source_content_hash: sha256:8df166c86a7ac0f01dc6e935261157c100a0d63ab49520c875eda82ef0b67352
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/AluruLab/MCPNet
  tools:
  - C++
  - MCPNet
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# aupr-auroc-network-evaluation

## Summary

Evaluate reconstructed gene regulatory/network inference quality by computing AUPR and AUROC against a ground-truth network matrix using MCPNet's au_pr_roc binary or the mcpnet pipeline's built-in evaluation triggered by the -x parameter.

## When to use

Use when a reconstructed GRN or GCN (e.g., MCP score matrix like gnw2000.mcp4.h5, or output from Pearson/MI/DPI) must be scored for network quality against a known ground-truth edge list (e.g., gnw2000_truenet.csv), or when comparing AUPRC across methods like MCPNet, ARACNe-AP, Arboreto, and Inferelator.

## When NOT to use

- No ground-truth network exists for the dataset (e.g., real yeast RNA-seq data without a validated edge list) — AUPR/AUROC cannot be computed; au_pr_roc requires a ground truth matrix via -x
- Input is a raw gene expression matrix rather than an inferred interaction/score matrix — infer the network first (mi, mcp, pearson) before evaluation
- Evaluating the ensemble method with only partial truth is needed as a training/testing split — use eval_mcpnet instead of plain au_pr_roc

## Inputs

- Inferred network matrix (HDF5 .h5, e.g., gnw2000.mcp4.h5 from the mcp binary, or Pearson/MI/DPI matrix)
- Ground-truth network matrix as CSV (e.g., gnw2000_truenet.csv)
- Optional transcription factor list (restricts evaluation to TF-to-gene interactions)
- Expression matrix (exp/csv/h5, e.g., gnw2000.h5) when using the full mcpnet pipeline with -x

## Outputs

- AUPR (AUPRC) value for the inferred network against the ground truth
- AUROC value for the inferred network against the ground truth
- Per-method AUPR/AUROC output files under the -o prefix when mcpnet is run with multiple -m methods

## How to apply

Run the au_pr_roc binary on the inferred network (-i, e.g., gnw2000.mcp4.h5) with the ground truth supplied via -x (gnw2000_truenet.csv); it computes the AUC of the PR and ROC curves against that matrix. Alternatively, the mcpnet primary executable performs MI computation, MCP score calculation, and AUPR/AUROC evaluation in one pass when -x is given; because -o takes a prefix, per-method output files (e.g., for -m 1 2 3) are generated under that prefix. If a list of transcription factors is supplied, only TF-to-gene interactions are evaluated. For ensemble evaluation with only partial ground truth on synthetic data, use eval_mcpnet, which splits the ground truth into training and testing sets and repeatedly evaluates the AUPR distribution. Input and output formats are selected by file extension (exp, csv, or h5).

## Related tools

- **MCPNet** (au_pr_roc computes AUC for PR and ROC curves against a ground-truth matrix; mcpnet runs MI computation, MCP score calculation, and AUPR/AUROC evaluation in one pass when -x is supplied) — https://github.com/AluruLab/MCPNet
- **C++** (Implementation language of the MCPNet binaries (au_pr_roc, mcpnet) that perform the evaluation) — https://github.com/AluruLab/MCPNet

## Examples

```
{MCPNet_build_dir}/bin/au_pr_roc -i gnw2000.mcp4.h5 -x {MCPNet_source_dir}/data/gnw2000_truenet.csv
```

## Evaluation signals

- AUPR and AUROC metrics are produced for each requested method (e.g., -m 1 2 3) when ground truth is supplied via -x, and per-method output files exist under the -o prefix
- All reported AUPRC/AUROC values are finite numbers in [0, 1]
- For GRN-vs-GCN comparisons, reconstructed GRN AUPRC exceeds its GCN counterpart (per Table 6, e.g., MCPNet vs. its GCN on gnw2000)
- Method rankings on gnw2000_truenet.csv reproduce paper expectations: MCPNet AUPRC beats other software while running in seconds (38 s) rather than hours (Arboreto ~29 h)
- eval_mcpnet runs yield a distribution of AUPR over the user-specified number of training/testing split iterations

## Limitations

- au_pr_roc requires a complete ground-truth matrix, so it is suitable mainly for synthetic expression data; partial truth scenarios need eval_mcpnet's train/test splitting
- If a transcription factor list is supplied, only TF-to-gene interactions are evaluated, so gene-gene (GCN) edges are excluded from scoring
- Metrics reflect the supplied ground truth only; results on gnw2000 are tied to that specific ground-truth file (gnw2000_truenet.csv)

## Evidence

- [methods] mcpnet pipeline performs MI computation, MCP score calculation, and AUPR/AUROC evaluation: "the primary executable that runs the MCPNet pipeline, including MI computation, MCP score calculation, and AUPR and AUROC metrics if groundtruth is supplied"
- [readme] au_pr_roc computes PR/ROC AUC against ground truth: "`au_pr_roc`: computes the AUC for PR and ROC curves based on a ground truth matrix."
- [readme] TF list restricts evaluation to TF-to-gene interactions: "If a list of transcription factors are supplied, then only TF to gene interactions are evaluated."
- [abstract] MCPNet AUPRC quality and speed advantage demonstrated: "we demonstrate that MCPNet produces better quality networks as measured by AUPRC, is significantly faster than all other gene network reconstruction software"
- [readme] eval_mcpnet evaluates ensemble AUPR over training/testing splits: "The groundtruth is divided into training and testing sets and the AUPR is evaluated for as many iterations as the user specifies."
