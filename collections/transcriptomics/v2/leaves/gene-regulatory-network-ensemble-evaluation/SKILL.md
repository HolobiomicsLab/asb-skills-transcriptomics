---
name: gene-regulatory-network-ensemble-evaluation
description: Use when you have a full synthetic/simulated groundtruth network plus
  expression data (HDF5/CSV/EXP) and need to test whether ensemble GRN coefficients
  remain reliable when optimized on only 0.5%-10% of the truth.
when_to_use_negative:
- Full groundtruth unsupervised evaluation — use `mcpnet` or `au_pr_roc` directly
  instead of the train/test ensemble protocol
- Input expression data is not in a supported format (HDF5, EXP, CSV) — convert first
  with the `convert` utility
- No gold-standard network exists at all — semi-supervised ensemble evaluation requires
  a groundtruth to split into training/testing sets
edam_operation: http://edamontology.org/operation_0279
edam_topics:
- http://edamontology.org/topic_0602
tools:
- name: MCPNet
  role: provides `eval_mcpnet`, the binary that performs train/test splitting of the
    groundtruth and captures the AUPR distribution over user-specified iterations;
    also supplies `mi`, `mcp`, and `transform` for candidate score matrices
  repo: https://github.com/AluruLab/MCPNet
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad373
    title: 'MCPNet: a parallel maximum capacity-based genome-scale gene network construction
      framework'
  source_content_hash: sha256:64cd7f3e430c580766fb1516e5ada54620f7da6f32d7f7ce72569e701b974a35
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/AluruLab/MCPNet
  tools:
  - MCPNet
  edam_operation: http://edamontology.org/operation_0279
  edam_topics:
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# gene-regulatory-network-ensemble-evaluation

## Summary

Benchmarks MCPNet's semi-supervised ensemble method by repeatedly splitting a gold-standard interaction network into training/testing sets, optimizing ensemble combination coefficients on the training subset, and capturing the distribution of held-out AUPRC values — used to show the ensemble is robust when only a small partial groundtruth (0.5%-10%) is available.

## When to use

Use when you have a full synthetic/simulated groundtruth network plus expression data (HDF5/CSV/EXP) and need to test whether ensemble GRN coefficients remain reliable when optimized on only 0.5%-10% of the truth.

## When NOT to use

- Full groundtruth unsupervised evaluation — use `mcpnet` or `au_pr_roc` directly instead of the train/test ensemble protocol
- Input expression data is not in a supported format (HDF5, EXP, CSV) — convert first with the `convert` utility
- No gold-standard network exists at all — semi-supervised ensemble evaluation requires a groundtruth to split into training/testing sets

## Inputs

- noise-free simulated yeast gene expression matrix (HDF5)
- yeast gold-standard interaction network (CSV, passed via -x)
- candidate method score matrices computed with MCPNet `mi`/`mcp` binaries and Stouffer/CLR transforms via `transform`
- gold-standard fraction levels (0.5%, 1%, 2%, 5%, 10%) and iteration count (100)

## Outputs

- per-iteration held-out test-AUPRC distributions for GCN and GRN ensembles at each gold-standard fraction
- mean and standard deviation summary table (e.g. yeast_ensemble_auprc_distributions.csv)

## How to apply

Compute candidate method score matrices with the MCPNet binaries (`mi`, `mcp` with specified `-m` method codes and `-t` thread count, plus Stouffer/CLR transforms via `transform`), then run `eval_mcpnet`, which takes essentially the same parameters as `mcpnet` (including `-x` for the groundtruth and `--mi-method`). At each gold-standard fraction (0.5%, 1%, 2%, 5%, 10%), the groundtruth is divided into training and testing sets; optimize the ensemble combination coefficients on the training subset, score the held-out testing subset with the combined ensemble ranking, and record the AUPRC for each iteration (the yeast experiment used 100 iterations per fraction). Aggregate the per-iteration test-AUPRC values into means and standard deviations for both the GCN and GRN ensembles at each fraction, and verify the means fall within the reported ranges (GCN 0.4135-0.4201; GRN 0.4979-0.5070, vs. 0.4198 with the full groundtruth).

## Related tools

- **MCPNet** (provides `eval_mcpnet`, the binary that performs train/test splitting of the groundtruth and captures the AUPR distribution over user-specified iterations; also supplies `mi`, `mcp`, and `transform` for candidate score matrices) — https://github.com/AluruLab/MCPNet

## Examples

```
{MCPNet_build_dir}/bin/eval_mcpnet -i {MCPNet_source_dir}/data/gnw2000.h5 -o gnw2000 -m 1 2 3 -t 4 --mi-method 1 -x {MCPNet_source_dir}/data/gnw2000_truenet.csv
```

## Evaluation signals

- GCN ensemble mean test-AUPRC across the 0.5%-10% training splits falls in 0.4135-0.4201
- GRN ensemble mean test-AUPRC falls in 0.4979-0.5070
- Mean AUPRC at partial fractions is close to the 0.4198 AUPRC obtained with the full groundtruth
- Each fraction's distribution is built from 100 independent train/test iterations
- Summary table is written with means and standard deviations per fraction and ensemble type

## Limitations

- Designed for the case where full groundtruth is available (synthetic/simulated expression data); it is suitable for partial-truth evaluation precisely because the full truth can be partitioned
- Requires the ensemble binaries and method codes to be compiled from source (C++ with CMake; OpenMP/MPI/HDF5/SIMD build options)
- The AUPR distribution depends on the number of user-specified iterations; fewer iterations give noisier distribution estimates

## Evidence

- [readme] eval_mcpnet executable for partial-truth ensemble evaluation: "eval_mcpnet`:  this executable is used for evaluating the performance of the ensemble method when only partial truth is provided."
- [methods] Groundtruth train/test splitting and AUPR over iterations: "The groundtruth is divided into training and testing sets and the AUPR is evaluated for as many iterations as the user specifies."
- [readme] eval_mcpnet shares parameters with mcpnet: "This binary has essentially the same parameters as mcpnet."
- [readme] AUPR distribution capture: "The distribution of the AUPR is captured and can be plotted."
- [other] Robustness finding for partial groundtruth training: [evidence span withheld — anchor status: not_found]
