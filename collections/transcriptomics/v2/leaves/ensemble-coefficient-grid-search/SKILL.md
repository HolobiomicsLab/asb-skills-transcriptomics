---
name: ensemble-coefficient-grid-search
description: Use when you have multiple MCP path-capacity score matrices (2-, 3-,
  and 4-path) plus partial groundtruth interactions, and want the optimal ensemble
  weighting that maximizes AUPRC.
when_to_use_negative:
- Full groundtruth is unavailable and only unsupervised reconstruction is possible
  — eval_mcpnet requires partial truth to split into training/testing sets
- You only have a single path-capacity matrix (no 2-, 3-, 4-path combination to weight)
  — use 'au_pr_roc' for direct AUPR/AUROC evaluation instead
- Input is raw gene expression rather than computed MCP score matrices — run the 'mi'
  and 'mcp' executables first
tools:
- name: MCPNet
  role: Provides the eval_mcpnet binary that performs the ensemble coefficient grid
    search, plus the mi and mcp executables that produce the 2-, 3-, 4-path capacity
    matrices being weighted
  repo: https://github.com/AluruLab/MCPNet
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btad373
    title: 'MCPNet: a parallel maximum capacity-based genome-scale gene network construction
      framework'
  source_content_hash: sha256:250a027b39a68b9474c90dc103ed99dda197293e58dced9f2b2bceef848310a8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/AluruLab/MCPNet
  tools:
  - MCPNet
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# ensemble-coefficient-grid-search

## Summary

Grid-search optimization of weighting coefficients that combine MCPNet's fixed-length (2-, 3-, 4-path) capacity score matrices into a single ensemble (Equation 4) MCP score MstL = Wst/(a2*ηst2 + a3*ηst3 + ... + aL*ηstL), with coefficients sweeping at 0.1 intervals and selected by maximal AUPRC on a held-out split of partial groundtruth.

## When to use

Use when you have multiple MCP path-capacity score matrices (2-, 3-, and 4-path) plus partial groundtruth interactions, and want the optimal ensemble weighting that maximizes AUPRC.

## When NOT to use

- Full groundtruth is unavailable and only unsupervised reconstruction is possible — eval_mcpnet requires partial truth to split into training/testing sets
- You only have a single path-capacity matrix (no 2-, 3-, 4-path combination to weight) — use 'au_pr_roc' for direct AUPR/AUROC evaluation instead
- Input is raw gene expression rather than computed MCP score matrices — run the 'mi' and 'mcp' executables first

## Inputs

- MCPNet MI matrix (HDF5, e.g. gnw2000.mi.h5) as input to 'mcp' for fixed-length path capacity scores
- Fixed-length (2-, 3-, 4-path) MCP capacity score matrices (HDF5, e.g. gnw2000.mcp4.h5)
- Partial groundtruth interaction network (CSV, e.g. gnw2000_truenet.csv) supplied via -x

## Outputs

- Optimal ensemble coefficient vector (weights summing to 1.0, selected at 0.1 grid interval by maximal AUPRC)
- Combined ensemble M4 score matrix written as valid HDF5 (.h5)
- Distribution of AUPR values across evaluation iterations (captured for plotting)

## How to apply

First compute MI and path capacities with MCPNet: run the 'mi' executable on the expression HDF5 (e.g. gnw2000.h5, -m 2, -t 4) and then 'mcp' on the MI matrix to get the fixed-length path (2, 3, 4) capacity scores. Invoke the 'eval_mcpnet' binary — which shares its parameters with mcpnet — supplying the groundtruth network (e.g. gnw2000_truenet.csv) via the -x flag so the ensemble can be scored against partial truth. eval_mcpnet divides the groundtruth into training and testing sets and evaluates the AUPR over the user-specified number of iterations. Sweep all weighting coefficient combinations at interval 0.1 over the 2-, 3-, and 4-path capacity matrices, recording the AUPRC of each combination on the held-out split. Select the coefficient vector with maximal AUPRC and emit it along with the combined M4 matrix; name outputs with the .h5 extension so MCPNet writes HDF5.

## Related tools

- **MCPNet** (Provides the eval_mcpnet binary that performs the ensemble coefficient grid search, plus the mi and mcp executables that produce the 2-, 3-, 4-path capacity matrices being weighted) — https://github.com/AluruLab/MCPNet

## Examples

```
{MCPNet_build_dir}/bin/mcpnet -i {MCPNet_source_dir}/data/gnw2000.h5 -o gnw2000 -m 1 2 3 -t 4 --mi-method 1 -x {MCPNet_source_dir}/data/gnw2000_truenet.csv
```

## Evaluation signals

- The emitted coefficient vector must correspond to the grid combination (0.1 interval) achieving the highest AUPRC across the specified evaluation iterations
- The combined M4 matrix must be written as a valid HDF5 file readable by the MCPNet utilities (e.g. convertible via the 'convert' binary)
- Coefficients of the optimal vector must sum to 1.0 as required by Equation 4
- AUPR distribution over iterations should be captured and plottable, confirming the train/test splitting ran for the user-specified iteration count

## Limitations

- The ensemble method is semi-supervised and requires at least partial groundtruth; it is suitable when full groundtruth is available, hence synthetic expression data
- The AUPRC estimate is distributional over train/test splits of partial truth, so a single-iteration result may not be stable; multiple iterations should be specified
- Default grid interval of 0.1 fixes the resolution of coefficient optimization; finer weighting resolution would require a different sweep

## Evidence

- [methods] ensemble multipath coefficient grid-search optimizer: [evidence span withheld — anchor status: not_found]
- [readme] ensemble evaluation with partial truth: "eval_mcpnet:  this executable is used for evaluating the performance of the ensemble method when only partial truth is provided."
- [readme] train/test AUPR iteration scheme: "The groundtruth is divided into training and testing sets and the AUPR is evaluated for as many iterations as the user specifies."
- [methods] groundtruth supply via -x: "`{MCPNet_build_dir}/bin/au_pr_roc -i gnw2000.mcp4.h5 -x {MCPNet_source_dir}/data/gnw2000_truenet.csv`"
- [readme] HDF5 output naming convention: "input and output format are specified via file extension, `exp`, `csv`, or `h5`."
